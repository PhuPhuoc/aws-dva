# Giải thích bổ sung - Phần 1

> *Phần này chứa kiến thức thực tế từ kinh nghiệm triển khai AWS. Đánh dấu **NÊN** hoặc **KHÔNG NÊN** để dễ nhớ.*

---

## 1. Hạ tầng toàn cầu AWS

### Lựa chọn Region
#### Giải thích cốt lõi
Region là vị trí địa lý chứa hạ tầng AWS; đặt workload gần người dùng giảm độ trễ, còn lựa chọn dịch vụ, giá và yêu cầu lưu trú dữ liệu có thể khác theo Region. So với cố định một Region, cân nhắc các yêu cầu này ngay khi thiết kế giúp tránh phải di chuyển workload về sau.

#### NÊN làm (DO)
- Chọn Region **gần người dùng cuối nhất** để giảm độ trễ.
- Ở Việt Nam thường dùng Singapore (`ap-southeast-1`) hoặc Sydney (`ap-southeast-2`).
- Kiểm tra dịch vụ có sẵn trong Region trước khi thiết kế kiến trúc.

#### KHÔNG NÊN làm (DON'T)
- Chọn Region chỉ vì giá rẻ khi người dùng cuối ở xa → độ trễ cao.
- Ghi cứng tên Region trong code → khó di chuyển.

### Availability Zones
#### Giải thích cốt lõi
Availability Zone là một hoặc nhiều trung tâm dữ liệu tách biệt trong Region, có nguồn điện/mạng dự phòng; triển khai qua nhiều AZ giúp cô lập lỗi nhưng vẫn kết nối bằng mạng độ trễ thấp.

**Thực tế:**
- **Cập nhật 2026-11-06:** AWS mô tả các AZ là những địa điểm tách biệt đáng kể, có thể cách nhau tới khoảng 60 dặm (100 km), nhưng kết nối bằng mạng băng thông cao với độ trễ một chữ số mili giây. Không nên coi khoảng cách vài km hay độ trễ 1–2 ms là giá trị cố định. [AWS Availability Zones](https://docs.aws.amazon.com/whitepapers/latest/aws-fault-isolation-boundaries/availability-zones.html)

#### NÊN làm (DO)
- Luôn triển khai resources trên **ít nhất 2 AZ** để có tính sẵn sàng cao.
- Thiết kế ứng dụng có thể chịu lỗi tại 1 AZ mà không ảnh hưởng toàn hệ thống.

#### KHÔNG NÊN làm (DON'T)
- Không đặt toàn bộ workload production trong một AZ → sự cố tại AZ có thể làm ngừng toàn dịch vụ; phân bố các thành phần phù hợp qua nhiều AZ.
**LƯU Ý QUAN TRỌNG:**
- **Cập nhật 06/11/2026:** Tài liệu AWS hiện hành nêu mỗi Region có ít nhất 3 AZ. Số AZ khả dụng trong account có thể khác do AZ bị giới hạn hoặc chưa được mở; kiểm tra bằng `describe-availability-zones` thay vì giả định một Region chỉ có 2 AZ. [AWS Availability Zones](https://docs.aws.amazon.com/global-infrastructure/latest/regions/aws-availability-zones.html)

---

## 2. IAM

### Users và Groups
#### Giải thích cốt lõi
IAM Groups gom users để gán quyền theo vai trò; users có thể thuộc nhiều group hoặc không thuộc group nào. So với policy gắn trực tiếp từng user, quản lý qua group giúp sửa quyền nhất quán khi thành viên thay đổi.

#### NÊN làm (DO)
- Tạo một IAM user cho mỗi người → CloudTrail và audit có thể gắn hành động với người cụ thể; so với dùng chung account, dễ thu hồi quyền khi nhân sự đổi.
- Dùng Group để cấp quyền theo vai trò → thay đổi membership một lần thay vì sửa từng user.
- Áp dụng least privilege → giảm phạm vi thiệt hại so với cấp AdministratorAccess mặc định.

#### KHÔNG NÊN làm (DON'T)
- Không gắn policy trùng lặp trực tiếp lên từng user khi quyền theo vai trò dùng chung được → thay đổi dễ lệch và khó rà soát; quản lý qua Group, chỉ thêm policy riêng khi cần.
- Không dùng AdministratorAccess cho mọi việc → credential bị lộ sẽ có quyền rộng; cấp policy tối thiểu theo vai trò.

### Chính sách mật khẩu
#### Giải thích cốt lõi
IAM account password policy quy định độ dài, loại ký tự, thời hạn và lịch sử mật khẩu cho IAM users; đây là lớp bảo vệ credentials nhưng không thay thế MFA hoặc chính sách truy cập.

#### NÊN làm (DO)
- Cấu hình độ dài và ngăn dùng lại mật khẩu yếu → làm tăng chi phí đoán mật khẩu; so với chỉ yêu cầu ký tự phức tạp, độ dài và MFA bổ sung lớp bảo vệ khác nhau.
```json
{
  "MinimumPasswordLength": 14,
  "RequireSymbols": true,
  "RequireNumbers": true,
  "RequireUppercaseCharacters": true,
  "RequireLowercaseCharacters": true,
  "PasswordReusePrevention": 24,
  "ExpirePasswords": true,
  "MaxPasswordAge": 90
}
```
#### KHÔNG NÊN làm (DON'T)
- Không chỉ dựa vào password policy để bảo vệ account → mật khẩu bị lộ vẫn có thể bị dùng; bật MFA và ưu tiên cơ chế đăng nhập mạnh theo chính sách tổ chức.

### MFA
#### Giải thích cốt lõi
MFA yêu cầu thêm yếu tố ngoài mật khẩu (thiết bị hoặc key người dùng sở hữu), nên kẻ biết mật khẩu vẫn khó đăng nhập nếu không có yếu tố thứ hai. So với chỉ dùng mật khẩu, MFA giảm rủi ro chiếm account.

#### NÊN làm (DO)
- Bắt buộc MFA cho Root account, tài khoản Admin và môi trường production → các danh tính này có quyền cao; yếu tố thứ hai giảm rủi ro khi mật khẩu bị lộ.

**NÊN dùng:**
- **Hardware Key (YubiKey)** → dùng yếu tố sở hữu khó bị lừa qua phishing hơn SMS; phù hợp khi cần mức bảo vệ cao.
- **Authy** → thuận tiện khi cần khôi phục/sử dụng trên nhiều thiết bị; so với hardware key, cân nhắc khả năng bảo vệ và mất thiết bị.

#### KHÔNG NÊN làm (DON'T)
- Dùng SMS MFA → có thể bị tấn công đổi SIM.
- Đặt Virtual MFA trên cùng thiết bị với mật khẩu → mất cả hai cùng lúc.

### Access Keys
#### Giải thích cốt lõi
Access Key ID và Secret Access Key là credentials tĩnh cho API/CLI/SDK; IAM Role cấp temporary credentials tự luân chuyển cho workload AWS, nên thường an toàn hơn key lâu dài.

#### NÊN làm (DO)
- **Dùng IAM Role thay Access Key trên EC2/Lambda** → nhận temporary credentials được AWS quản lý thay vì tự lưu/luân chuyển key tĩnh.
- **Luân chuyển key tĩnh nếu không thể loại bỏ** → giới hạn thời gian một key bị lộ còn hiệu lực; ưu tiên role để tránh chu kỳ rotate thủ công.
- **Chỉ lưu Access Key khi thật sự bắt buộc** → Secrets Manager/Parameter Store cung cấp nơi kiểm soát truy cập tốt hơn file/code; nếu workload chạy trên AWS, dùng role thay vì lưu key.

#### KHÔNG NÊN làm (DON'T)
- Ghi cứng Access Key trong code → có thể làm lộ key.
- Lưu Access Key trong Git → lịch sử Git có thể giữ lại mãi mãi.
- Dùng Access Key của root account → hoàn toàn không an toàn.

### IAM Roles
#### Giải thích cốt lõi
IAM Role là danh tính có permissions mà AWS service hoặc workload có thể assume để nhận temporary credentials; khác với IAM user/access key tĩnh, role cho phép quyền được cấp theo phiên và quản lý tập trung.

**Khi nào sử dụng:**
- EC2 instance cần truy cập các dịch vụ AWS.
- Lambda function cần quyền.
- ECS task cần truy cập tài nguyên AWS.
- Truy cập giữa các tài khoản AWS.

#### NÊN làm (DO)
- Tạo role riêng cho mỗi Lambda function với quyền vừa đủ → lỗi hoặc lộ credential của một function không kéo theo quyền của function khác.
- Dùng IAM Role thay Access Key cho workload chạy trên AWS → nhận temporary credentials thay vì quản lý key tĩnh.
#### KHÔNG NÊN làm (DON'T)
- Không dùng chung một Role quá rộng cho nhiều workload → một workload bị xâm nhập có thể kế thừa toàn bộ quyền chung; tạo role riêng với least privilege cho từng nhu cầu.

### Tóm tắt thực hành tốt nhất về IAM

| Thực hành | Mức ưu tiên |
|-----------|----------|
| Bật MFA ở mọi nơi | Rất cao |
| Không dùng root account | Rất cao |
| Một user cho mỗi người | Cao |
| Dùng groups | Cao |
| Quyền tối thiểu | Cao |
| Kiểm tra thường xuyên | Trung bình |
| Luân chuyển Access Key | Trung bình |

---

## 3. EC2

### Instance Types - chọn loại nào?
#### Giải thích cốt lõi
**Instance Type** kết hợp CPU, bộ nhớ, network và đôi khi có local storage; chọn theo nút thắt của ứng dụng và yêu cầu độ bền dữ liệu. General Purpose cân bằng tài nguyên, còn các họ Compute/Memory/Storage Optimized ưu tiên một nhóm tài nguyên; Instance Store là tạm thời, khác với EBS.

#### NÊN làm (DO)
- So sánh instance families theo workload thực tế → chọn CPU, RAM và storage vừa đủ thay vì dựa riêng vào giá.

#### KHÔNG NÊN làm (DON'T)
- Không chọn instance chỉ vì có nhiều CPU hoặc RAM → tài nguyên không dùng vẫn tốn phí; benchmark workload và chọn family phù hợp.

| Trường hợp sử dụng | NÊN DÙNG | KHÔNG NÊN |
|----------|----------|-----------|
| Máy chủ web nhỏ | t3.micro, t3.small | c5.large (quá dư cấu hình) |
| Máy chủ web lớn | m5.large, m5.xlarge | t3.micro (quá yếu) |
| Xử lý theo lô | c5.2xlarge, c5.4xlarge | r5.large (lãng phí RAM) |
| Cơ sở dữ liệu | r5.large, r5.xlarge | t3.micro (không đủ RAM) |
| Cache (Redis) | r5b (EBS) hoặc im4gn (NVMe cục bộ) | c5 (không có SSD cục bộ) |
| Lưu trữ tệp | i3 (NVMe cục bộ) | r5 (không cần thêm RAM) |

### General Purpose (T, M series)
#### Giải thích cốt lõi
General Purpose cân bằng CPU, RAM và network; T series dùng CPU credits để burst trong lúc nhu cầu tăng, còn M series phù hợp mức tải ổn định hơn. So với T, M tránh giới hạn do credits nhưng có thể tốn hơn cho tải thấp/ngắt quãng.


**T series (có khả năng burst):**
- Phù hợp: phát triển/kiểm thử, web nhỏ, cơ sở dữ liệu nhỏ.
- **Hệ thống credit:** có CPU credits để tăng năng lực xử lý khi cần.
- **NÊN:** dùng cho khối lượng công việc không liên tục, có mức sử dụng cơ sở thấp.
- **KHÔNG NÊN:** dùng cho môi trường production có khối lượng công việc ổn định 24/7 → CPU credits cạn và hiệu năng bị giới hạn.

**M series:**
- Cân bằng giữa năng lực tính toán và bộ nhớ.
- Phù hợp với ứng dụng web mục đích chung và cơ sở dữ liệu vừa; chọn M series cho tải ổn định cần cân bằng CPU/RAM thay vì T series khi không muốn phụ thuộc burst credits.
#### KHÔNG NÊN làm (DON'T)
- Không dùng T series cho tải CPU ổn định 24/7 → credits có thể cạn và hiệu năng bị giới hạn; chọn M hoặc family phù hợp hơn với mức tải đo được.

### Tối ưu cho tính toán (C series)
#### Giải thích cốt lõi
Compute Optimized dành cho workload bị giới hạn bởi CPU; so với General Purpose, nó tăng tỷ lệ compute nhưng không phải lựa chọn mặc định cho ứng dụng cần nhiều RAM hoặc storage.

#### NÊN làm (DO)
- Dùng C series cho batch processing, mã hóa media hoặc HPC cần CPU cao → tận dụng tỷ lệ compute lớn.

#### KHÔNG NÊN làm (DON'T)
- Không chọn C series cho database/cache bị giới hạn bởi RAM hoặc storage → tài nguyên chính vẫn là nút thắt; cân nhắc R/X hoặc I/D/H theo workload.

- Phù hợp: xử lý theo lô, mã hóa video, suy luận ML.
- Khả năng có **Instance Store** phụ thuộc từng **Instance Type**; một số biến thể có ổ cục bộ, còn nhiều loại không có. Kiểm tra cấu hình của instance cụ thể; nếu không có Instance Store thì dùng EBS.

### Tối ưu cho bộ nhớ (R, X series)
#### Giải thích cốt lõi
Memory Optimized có tỷ lệ RAM/CPU cao, phù hợp dataset lớn nằm trong memory; so với Compute Optimized, nhóm này dành nhiều capacity hơn cho bộ nhớ.

#### NÊN làm (DO)
- Chọn R/X khi database hoặc cache cần nhiều RAM → giảm thiếu bộ nhớ và paging.

#### KHÔNG NÊN làm (DON'T)
- Không mua dư RAM khi workload không giữ dataset trong memory → lãng phí capacity; chọn General Purpose hoặc đúng Instance Type sau khi đo.

- Phù hợp: cơ sở dữ liệu lớn, cache trong bộ nhớ, SAP HANA.
- **R6i, R5**: RAM DDR4.

### Tối ưu cho lưu trữ (I, D, H series)
#### Giải thích cốt lõi
Storage Optimized ưu tiên thông lượng/IOPS hoặc ổ cục bộ cho dataset lớn; khả năng và loại Instance Store phụ thuộc từng Instance Type, còn EBS cung cấp block storage bền vững qua mạng.

#### NÊN làm (DO)
- Dùng I/D/H khi workload cần lưu trữ cục bộ hoặc truy cập tuần tự lớn; kiểm tra loại ổ cụ thể trước khi chọn.

#### KHÔNG NÊN làm (DON'T)
- Không đặt dữ liệu duy nhất cần giữ trên Instance Store → dữ liệu mất theo vòng đời instance; dùng EBS hoặc dịch vụ lưu trữ bền vững.

| Series | Loại lưu trữ | Trường hợp sử dụng |
|--------|-------------|----------|
| **I3** | NVMe SSD | Cơ sở dữ liệu, NoSQL, cache |
| **I4i** | NVMe SSD | Cơ sở dữ liệu cần hiệu năng cao hơn |
| **D2** | HDD | HDFS, MapReduce |
| **H1** | HDD | Big data, kho dữ liệu |

**NÊN:** Dùng I series cho cơ sở dữ liệu cần IOPS cao.

### Script User Data
#### Giải thích cốt lõi
EC2 User Data truyền script bootstrap khi launch instance; mặc định script chạy trong lần boot đầu tiên và với quyền root. So với cấu hình thủ công, cách này lặp lại được cho instance mới nhưng không thay thế công cụ quản lý cấu hình dài hạn.

#### NÊN làm (DO)
- Dùng User Data cho cài đặt/thiết lập ban đầu và giữ script idempotent → instance mới được cấu hình nhất quán.

#### KHÔNG NÊN làm (DON'T)
- Không dựa vào User Data mặc định để chạy tác vụ mỗi lần reboot → script không tự chạy lại; dùng systemd hoặc Systems Manager State Manager/Run Command khi cần chạy lặp.

**Khi nào sử dụng:**
- Cài đặt các gói phần mềm khi khởi chạy.
- Thiết lập ứng dụng ban đầu.
- Tải mã nguồn từ GitHub/S3.

**Ví dụ:**
```bash
#!/bin/bash
yum update -y
yum install -y httpd php php-mysql
systemctl start httpd
systemctl enable httpd
```

**LƯU Ý:**
- Theo mặc định, script chạy một lần ở lần boot đầu tiên của instance.
- Reboot hoặc stop/start không tự chạy lại User Data theo mặc định.
- Nếu cần quản lý cấu hình lặp lại hoặc chạy lệnh về sau → dùng cloud-init, Systems Manager State Manager hoặc Run Command tùy nhu cầu.

### Security Groups
#### Giải thích cốt lõi
Security Group là firewall stateful gắn với ENI/instance, chỉ có rule Allow; mặc định chặn inbound và cho phép outbound. Có thể tham chiếu CIDR hoặc Security Group khác, khác với Network ACL áp dụng ở subnet.


**NGUYÊN TẮC VÀNG:**

1. **Chỉ mở port cần thiết**
   - SSH (22): chỉ từ địa chỉ IP tin cậy hoặc Bastion Host.
   - HTTP (80) và HTTPS (443): mở ra Internet nếu dịch vụ thực sự công khai.
   - Các cổng database (3306, 5432): chỉ cho phép từ Security Group của máy chủ ứng dụng.

2. **Tham chiếu Security Group thay vì IP khi phù hợp**
   - Dùng `sg-0123456789abcdef0` (SG của máy chủ ứng dụng) làm nguồn/đích thay vì dải IP cố định.
   - Khi IP của instance thay đổi, quy tắc tham chiếu SG vẫn áp dụng cho các ENI mang SG đó.

3. **Phân tầng Security Groups**
   ```
   Internet → ALB Security Group (port 80, 443)
   ALB → SG của máy chủ web (chỉ từ ALB SG)
   Máy chủ web → SG của máy chủ ứng dụng (chỉ từ SG máy chủ web)
   Máy chủ ứng dụng → SG cơ sở dữ liệu (chỉ từ SG máy chủ ứng dụng)
   ```

#### NÊN làm (DO)
- Tạo Security Group riêng cho từng tầng → giới hạn đường truy cập giữa web/app/database và dễ audit hơn một SG dùng chung.
- Dùng tên rõ ràng, ví dụ `sg-webservers-prod`, `sg-databases-prod` → người vận hành dễ nhận biết phạm vi trước khi sửa rule.

#### KHÔNG NÊN làm (DON'T)
- Mở SSH từ `0.0.0.0/0` → tăng bề mặt tấn công; thay vào đó giới hạn IP hoặc dùng kết nối quản trị phù hợp.
- Gắn nhiều SG trùng lặp vào resources → khó kiểm tra quyền hiệu lực.

### Các cổng phổ biến
#### Giải thích cốt lõi
Security Group chỉ điều khiển traffic theo protocol/port và source/destination; mở port công khai chỉ cần cho dịch vụ thật sự public. Database nên nhận traffic từ app Security Group thay vì Internet.

#### NÊN làm (DO)
- Giới hạn SSH và database ports theo nguồn tin cậy → giảm bề mặt tấn công so với mở toàn Internet.

#### KHÔNG NÊN làm (DON'T)
- Không mở port quản trị hoặc database cho `0.0.0.0/0` → bất kỳ client nào cũng có thể thử kết nối; giới hạn IP/SG hoặc dùng phương thức quản trị riêng.

| Cổng | Quy tắc Security Group |
|------|-------------------|
| 22 (SSH) | Chỉ từ địa chỉ IP của bạn hoặc bastion |
| 80 (HTTP) | `0.0.0.0/0` nếu website công khai |
| 443 (HTTPS) | `0.0.0.0/0` nếu website công khai |
| 3306 (MySQL) | Chỉ từ SG của máy chủ ứng dụng |
| 5432 (PostgreSQL) | Chỉ từ SG của máy chủ ứng dụng |
| 6379 (Redis) | Chỉ từ SG của máy chủ ứng dụng (mạng riêng) |
| 27017 (MongoDB) | Chỉ từ SG của máy chủ ứng dụng (mạng riêng) |

### EC2 Instance Connect
#### Giải thích cốt lõi
EC2 Instance Connect gửi khóa SSH tạm thời để mở phiên trình duyệt/SSH mà không cần phân phối key pair lâu dài; instance vẫn cần hệ điều hành được hỗ trợ, IAM permission và đường mạng cho SSH.

#### NÊN làm (DO)
- Dùng Instance Connect khi cần truy cập tương tác mà không muốn quản lý key file lâu dài; giới hạn SSH bằng Security Group và cấp IAM permission phù hợp.

#### KHÔNG NÊN làm (DON'T)
- Không giả định Instance Connect bỏ yêu cầu mạng hoặc port 22 → phiên kết nối vẫn thất bại nếu không có route/ingress; với instance private, dùng đường kết nối private/Bastion hoặc Systems Manager Session Manager.

**Ưu điểm:**
- Không cần key pair.
- Terminal chạy trên trình duyệt.
- Có thể kiểm tra ai đã SSH.

**Điều kiện:**
- Instance phải có đường mạng phù hợp: public route hoặc kết nối private qua **EC2 Instance Connect Endpoint**.
- Target Security Group phải cho phép SSH qua port 22 từ nguồn phù hợp; quyền IAM cũng phải cho phép kết nối.
### Các lựa chọn mua - So sánh thực tế
#### Giải thích cốt lõi
EC2 purchase options đổi mức giá lấy độ linh hoạt hoặc khả năng có capacity: On-Demand không có cam kết; Reserved/Savings Plans chiết khấu khi workload ổn định; Spot dùng capacity dư thừa nên có thể bị gián đoạn; Dedicated Host/Capacity Reservation giải quyết nhu cầu phần cứng hoặc AZ cụ thể.

#### On-Demand
#### Giải thích cốt lõi
On-Demand không yêu cầu cam kết trước và tính phí theo mức sử dụng; so với RI/Savings Plans, cách này linh hoạt nhưng thường kém tiết kiệm hơn cho tải ổn định.


#### NÊN làm (DO)
- Dùng On-Demand cho dev/test hoặc workload ngắn, khó dự đoán → không phải cam kết trước; so với RI/Savings Plans linh hoạt hơn khi usage chưa ổn định.
- Dùng lần triển khai đầu để đo nhu cầu → dữ liệu usage giúp quyết định commitment sau này.
#### KHÔNG NÊN làm (DON'T)
- Không chọn On-Demand cho tải ổn định lâu dài nếu mục tiêu là giảm chi phí → mức giá/giờ cao hơn; đo usage rồi so sánh Reserved Instances hoặc Savings Plans.

#### Reserved Instances
#### Giải thích cốt lõi
Reserved Instance là cam kết sử dụng cấu hình trong thời hạn để nhận chiết khấu; Regional scope linh hoạt hơn nhưng không dự trữ capacity, còn Zonal scope có thể dự trữ capacity trong AZ.


#### NÊN làm (DO)
- Dùng Reserved Instances cho database production 24/7 hoặc workload ổn định → cam kết thời hạn đổi lấy chiết khấu so với On-Demand.
- Chọn Regional RI nếu cần tính linh hoạt theo Region; chọn Zonal RI khi cần giữ capacity trong AZ cụ thể.

**TIẾT KIỆM:** 72% so với On-Demand (3 năm, trả trước toàn bộ).

**LƯU Ý QUAN TRỌNG:**
- RI không đảm bảo capacity → có thể không có instance khả dụng.
- **Zonal RI** → dự trữ capacity trong AZ cụ thể.
- **Regional RI** → không dự trữ capacity, chỉ giảm giá.
#### KHÔNG NÊN làm (DON'T)
- Không coi Regional RI là bảo đảm có instance capacity → chỉ Zonal RI dự trữ capacity trong AZ; nếu cần capacity, chọn Zonal RI hoặc Capacity Reservation.

#### Savings Plans
#### Giải thích cốt lõi
Savings Plans giảm giá bằng cam kết chi tiêu theo giờ; EC2 Instance Savings Plans giới hạn trong family/Region, còn Compute Savings Plans linh hoạt hơn qua family/Region và loại compute đủ điều kiện. So với RI, đổi lại khả năng linh hoạt là vẫn phải trả phần cam kết.


**Phạm vi linh hoạt:**
- **EC2 Instance Savings Plans** áp dụng cho một instance family trong một Region, linh hoạt giữa kích cỡ, OS và tenancy trong family đó.
- **Compute Savings Plans** linh hoạt hơn: áp dụng giữa instance family, Region, OS, tenancy và các tùy chọn capacity đủ điều kiện.
- Ví dụ: mức cam kết cho M5 ở một Region có thể áp dụng cho kích cỡ M5 khác; khoản cam kết vẫn tính theo giờ kể cả khi mức sử dụng thấp hơn.

#### NÊN làm (DO)
- Dùng Savings Plans khi usage ổn định và có thể ước tính chi tiêu theo giờ → có chiết khấu mà linh hoạt hơn RI khi instance size/type thay đổi; chọn Compute Savings Plans nếu cần linh hoạt qua family/Region.

**Cập nhật 2026-11-06:** Kiểm tra loại plan và độ linh hoạt cần thiết trong tài liệu [AWS Savings Plans](https://docs.aws.amazon.com/savingsplans/latest/userguide/what-is-savings-plans.html) trước khi cam kết.
#### KHÔNG NÊN làm (DON'T)
- Không cam kết mức cao hơn usage ổn định dự kiến → phần cam kết không dùng hết vẫn bị tính; ước lượng usage theo giờ và chọn loại plan theo mức linh hoạt cần thiết.

#### Spot Instances
#### Giải thích cốt lõi
Spot sử dụng EC2 capacity dư thừa với giá thấp hơn On-Demand nhưng AWS có thể thu hồi instance; checkpoint, retry hoặc phân phối workload giúp chịu được gián đoạn.


**Ưu điểm:**
- Giảm đến 90%.
- Rất phù hợp cho tác vụ xử lý theo lô và xử lý dữ liệu.

**NHƯỢC ĐIỂM:**
- **Có thể bị kết thúc bất cứ lúc nào** với 2 phút thông báo.
- Không đảm bảo tính sẵn sàng.

#### NÊN làm (DO)
- Dùng Spot cho batch có thể retry → tận dụng giá thấp dù instance có thể bị thu hồi.
- Dùng Spot cho Spark/Hadoop phân tán và CI build có thể chạy lại → chia nhỏ/checkpoint công việc để chịu interruption.
- Dùng Spot cho ML training có checkpoint → tiếp tục từ trạng thái đã lưu thay vì mất toàn bộ tiến độ.

#### KHÔNG NÊN làm (DON'T)
- Không dùng Spot cho database, production web hoặc workload liên tục không thể phục hồi → interruption có thể gây mất dữ liệu/gián đoạn dịch vụ; dùng On-Demand/commitment cho phần stateful và Spot cho phần retryable.

**Chiến lược Spot:**
```yaml
# Ví dụ: Auto Scaling Group với Spot
instanceTypes:
  - m5.large
  - m5.xlarge
  - m4.large
SpotAllocationStrategy: "lowest-price"
```

#### Dedicated Hosts
#### Giải thích cốt lõi
Dedicated Host dành riêng máy chủ vật lý cho account, giúp kiểm soát placement và hỗ trợ một số giấy phép theo socket/core; so với Dedicated Instance, Host cung cấp quyền kiểm soát phần cứng rõ hơn nhưng thường tốn kém hơn.


#### NÊN làm (DO)
- Chọn Dedicated Host khi cần license theo socket/core, placement control hoặc yêu cầu cô lập phần cứng → các nhu cầu này không được giải quyết tương tự bằng tenancy mặc định.
#### KHÔNG NÊN làm (DON'T)
- Không thuê Dedicated Host nếu không cần license/placement control → chi phí cao và capacity có thể khó bảo đảm; chọn tenancy mặc định hoặc Dedicated Instance nếu đáp ứng yêu cầu.
#### Capacity Reservations
#### Giải thích cốt lõi
Capacity Reservation giữ EC2 capacity On-Demand trong AZ cụ thể nhưng không giảm giá; khác với RI/Savings Plans, nó giải quyết khả năng khởi chạy chứ không phải chiết khấu.


#### NÊN làm (DO)
- Dùng Capacity Reservation khi phải bảo đảm khả năng khởi chạy trong AZ cụ thể, nhất là cho workload ngắn hạn/DR → khác với RI/Savings Plans, nó giữ capacity nhưng không giảm giá.
**LƯU Ý:** Không tiết kiệm gì so với On-Demand.
#### KHÔNG NÊN làm (DON'T)
- Không duy trì reservation khi không cần capacity → vẫn trả phí On-Demand; hủy reservation hoặc dùng RI/Savings Plans nếu mục tiêu là giảm giá.

### Lưu trữ EBS - Kinh nghiệm thực tế

#### Các loại Volume
#### Giải thích cốt lõi
EBS cung cấp block storage bền vững qua mạng; gp3/gp2 cân bằng giá và hiệu năng, io1/io2 cấp IOPS, còn st1/sc1 tối ưu thông lượng/lưu trữ lạnh. So với Instance Store, EBS giữ dữ liệu độc lập với vòng đời instance.

#### NÊN làm (DO)
- Ưu tiên gp3 khi đáp ứng hiệu năng vì giá thường thấp hơn gp2 và IOPS/thông lượng độc lập với dung lượng; dùng io2 khi cần PIOPS/độ bền cao hơn.

| Loại | Trường hợp sử dụng | NÊN/KHÔNG |
|------|----------|-----------|
| **gp3** | Mục đích chung (mặc định) | ✅ NÊN - giá rẻ nhất |
| **gp2** | Loại cũ, vẫn được hỗ trợ | ⚠️ Ưu tiên gp3 cho workload mới nếu phù hợp |
| **io1** | Cơ sở dữ liệu cần IOPS được cấp phát | ✅ Dùng khi cần PIOPS |
| **io2 Block Express** | Hiệu năng rất cao | ✅ SAP, Oracle |
| **st1** | HDD tối ưu thông lượng | ⚠️ Không dùng làm boot volume |
| **sc1** | Lưu trữ lạnh | ⚠️ Rất chậm |
#### KHÔNG NÊN làm (DON'T)
- Không dùng st1/sc1 làm boot volume hoặc chọn volume chỉ theo giá → không khởi động được OS hoặc không đạt IOPS cần thiết; dùng gp3/gp2 cho boot và io1/io2 khi cần PIOPS.

#### Thực hành tốt nhất với EBS
#### Giải thích cốt lõi
EBS volume type, size và IOPS quyết định chi phí/hiệu năng; mã hóa bảo vệ dữ liệu lưu trữ/truyền tải, snapshot tạo bản sao lưu. So với cấp dư ngay từ đầu, tăng dung lượng thường dễ hơn giảm dung lượng.


#### NÊN làm (DO)
- Dùng **gp3** làm mặc định cho workload phù hợp → thường rẻ hơn gp2 và tách IOPS/thông lượng khỏi dung lượng.
- Bật **EBS encryption**; nếu bật **EBS encryption by default** cho một Region, volume mới và snapshot copy mới trong Region đó sẽ được mã hóa.
- Tách volumes khi cần quản lý lifecycle/hiệu năng độc lập → dễ sao lưu hoặc thay đổi từng phần hơn một volume gộp.
- Tạo snapshot định kỳ → có thể phục hồi dữ liệu về điểm sao lưu; kiểm thử restore để xác nhận quy trình.

#### KHÔNG NÊN làm (DON'T)
- Cấp EBS quá lớn từ đầu → nếu cần giảm phải tạo volume mới và sao chép dữ liệu; bắt đầu vừa đủ rồi tăng dần.
- Dùng EBS cho **dữ liệu tạm thời** → nên dùng Instance Store.
- Không để **Delete on Termination** bật khi cần giữ volume sau khi terminate → root volume có thể bị xóa; tắt thuộc tính này hoặc tạo snapshot trước khi kết thúc instance.
- Không coi snapshot/volume mã hóa là thay thế cho chiến lược phục hồi đã kiểm thử → backup có thể không đáp ứng RPO/RTO; định kỳ kiểm tra restore và quản lý snapshot retention.

**VỀ KÍCH THƯỚC EBS:**
```
❌ KHÔNG NÊN: 1000 GB gp3 ngay từ đầu
✅ NÊN: 50 GB gp3, sau đó resize (dễ thực hiện khi hệ thống đang chạy)
```

**VỀ IOPS:**
- gp3: mức cơ sở 3000 IOPS, có thể tăng đến 16000.
- io2: độ bền 99.999%, tối đa 256000 IOPS.

#### Sao lưu EBS
#### Giải thích cốt lõi
EBS Snapshot lưu bản sao incremental của volume; Data Lifecycle Manager/AWS Backup tự động hóa lịch và retention. So với thao tác thủ công, policy giảm bỏ sót backup, còn sao chép Region hỗ trợ phục hồi khi Region nguồn gặp sự cố.

#### NÊN làm (DO)

**Phương pháp:**

1. **EBS Snapshots (khuyến nghị):**
   - Tự động hóa bằng Data Lifecycle Manager (DLM).
   - Hoặc dùng AWS Backup.
   - Sao chép giữa các Region để phục vụ DR.

2. **Snapshots tự động:**
```bash
# Tạo snapshot
aws ec2 create-snapshot --volume-id vol-1234567890abcdef0 --description "Daily backup"
```

3. **Với cơ sở dữ liệu:**
   - Dùng **RDS** thay vì EC2 + EBS cho cơ sở dữ liệu.
   - RDS tự động sao lưu và hỗ trợ khôi phục về một thời điểm cụ thể.
   - Triển khai Multi-AZ để có dự phòng theo thời gian thực.

#### KHÔNG NÊN làm (DON'T)
- Không bỏ qua kiểm tra khả năng restore hoặc lưu mọi snapshot vô thời hạn → backup có thể không phục hồi được và chi phí tăng; thử restore định kỳ và đặt retention theo yêu cầu.
### Kết nối SSH và Bastion Host
#### Giải thích cốt lõi
Bastion Host là máy trung gian để quản trị instance private qua SSH; Systems Manager Session Manager tạo phiên quản trị qua SSM mà không cần public IP, Bastion Host hoặc mở port 22.

**Vấn đề bảo mật trong thực tế:**

#### KHÔNG NÊN làm (DON'T)
```bash
# SSH trực tiếp vào production EC2
ssh -i key.pem ec2-user@public-ip
```
- Không SSH trực tiếp đến production instance qua public IP với key không được giới hạn → port công khai và key bị lộ làm tăng bề mặt tấn công; giới hạn nguồn và dùng Bastion Host hoặc Systems Manager Session Manager.

#### NÊN làm (DO) — Dùng Bastion Host khi cần điểm SSH trung gian:
```
Máy tính của nhà phát triển
    ↓ (SSH)
Bastion Host (1 instance, public IP, SG nghiêm ngặt)
    ↓ (SSH qua Systems Manager Session Manager)
EC2 riêng tư (không có public IP)
```

#### NÊN làm (DO) — Ưu tiên Systems Manager Session Manager khi phù hợp:
- Không cần bastion host.
- Không cần mở port 22.
- Có audit logs.
- Không cần key pair.

```bash
# Kết nối qua SSM
aws ssm start-session --target instance-id
```

### EC2 Placement Groups
#### Giải thích cốt lõi
Placement Group điều khiển cách EC2 instances được đặt trên hạ tầng: Cluster tối ưu kết nối độ trễ thấp trong một AZ, Partition tách nhóm trên nhiều rack, Spread cô lập instances trên phần cứng khác nhau.


#### NÊN làm (DO)
| Loại | Mô tả | Trường hợp sử dụng |
|------|--------|----------|
| **Cluster** | Instances trong cùng AZ | HPC, huấn luyện ML (độ trễ thấp) |
| **Partition** | Phân tán qua các partitions | HDFS, HBase, Cassandra |
| **Spread** | Phân tán trên phần cứng | Ứng dụng quan trọng |
- Chọn Cluster cho workload nhạy độ trễ, Partition/Spread khi cần cô lập lỗi → các loại placement group đánh đổi locality lấy fault isolation.

**LƯU Ý:**
- Cluster: có rủi ro - nếu phần cứng gặp lỗi thì tất cả đều lỗi.
- Spread: bảo đảm tính sẵn sàng cao.
#### KHÔNG NÊN làm (DON'T)
- Không chọn Cluster cho workload cần chống lỗi phần cứng trên nhiều AZ → instances trong group tập trung cùng AZ; dùng Partition/Spread hoặc Multi-AZ architecture khi ưu tiên fault isolation.

---

## 4. AWS CLI & SDK

### Thực hành tốt nhất với AWS CLI
#### Giải thích cốt lõi
AWS CLI dùng credential provider chain để xác thực API calls; named profiles lưu cấu hình cho nhiều tài khoản/Region, còn AWS IAM Identity Center hoặc role federation tránh quản lý access keys dài hạn.

#### NÊN làm (DO)
- Ưu tiên IAM Identity Center/named profiles cho người dùng và IAM Role cho workload → tách credentials theo account và tránh dùng chung access key dài hạn.

**Cài đặt:**
```bash
# Linux/macOS
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Verify
aws --version
```

**Cấu hình:**
```bash
aws configure
# AWS Access Key ID: AKIA...
# AWS Secret Access Key: ...
# Default region name: ap-southeast-1
# Default output format: json
```

**Sử dụng Profile:**
```bash
# Tạo named profile
aws configure --profile production

# Sử dụng
aws s3 ls --profile production
```
#### KHÔNG NÊN làm (DON'T)
- Không chia sẻ hoặc lưu access keys lâu dài trong máy/CI khi có thể dùng IAM Identity Center hoặc role credentials → key tĩnh khó thu hồi và có thể bị lộ; dùng named profiles cho môi trường phát triển và role/SSO cho workload.

### AWS SDK
#### Giải thích cốt lõi
AWS SDK gọi service APIs từ ứng dụng và dùng credential provider chain, nên code có thể lấy quyền từ IAM Role trên AWS hoặc profile khi phát triển cục bộ. So với tự ký request, SDK quản lý giao thức và xác thực cho ứng dụng.

#### NÊN làm (DO)

**Python (Boto3) - Phổ biến nhất:**
```python
import boto3

# Sử dụng default credentials
s3 = boto3.client('s3')
s3.list_buckets()

session = boto3.Session(profile_name='production')
dynamodb = session.client('dynamodb')

```

#### KHÔNG NÊN làm (DON'T)
- Ghi cứng credentials trong code → mã nguồn/log có thể làm lộ key; dùng credential provider chain.
- Commit credentials vào Git → key có thể còn trong lịch sử sau khi xóa; thu hồi/rotate key và dùng role/profile an toàn hơn.

#### NÊN làm (DO)
- Dùng IAM Role cho EC2/Lambda → AWS cung cấp temporary credentials mà không cần lưu key trong ứng dụng.
- Dùng named profile khi phát triển cục bộ → tách cấu hình theo môi trường; dùng AWS Secrets Manager chỉ khi cần lưu secret bên ngoài AWS.

---

## Tóm tắt các điểm cần nhớ

### Nên làm ngay:
1. ✅ Bật MFA cho root account.
2. ✅ Tạo IAM user riêng cho bản thân.
3. ✅ Không dùng root account cho công việc hằng ngày.
4. ✅ Dùng Security Groups đúng cách.
5. ✅ Sao lưu EBS volumes.

### Không nên:
1. ❌ Mở SSH từ `0.0.0.0/0`.
2. ❌ Ghi cứng credentials.
3. ❌ Cấp quá nhiều permissions.
4. ❌ Dùng Spot cho database.
5. ❌ Cấu hình EBS quá lớn ngay từ đầu.

## Cập nhật AWS (ngày kiểm tra: 2026-11-06)

### CloudFront và mạng Edge

#### Giải thích cốt lõi
Slide nguồn Trang 15 ghi 400+ Edge Locations và 13 Regional Caches. Theo trang tính năng CloudFront của AWS được kiểm tra ngày **2026-11-06**, CloudFront hiện công bố **750+ Points of Presence (PoPs)** tại **100+ thành phố, 50+ quốc gia**, **15 Regional Edge Caches**, và thêm **1.140+ embedded PoPs** tại hơn 300 thành phố. PoP, Regional Edge Cache và embedded PoP là các lớp hạ tầng khác nhau; không cộng các số này thành một loại điểm duy nhất. [Mạng edge toàn cầu CloudFront](https://aws.amazon.com/cloudfront/features/)

#### NÊN làm
- **Dùng số liệu trên trang CloudFront hiện hành khi mô tả phạm vi phủ sóng** → Vì số liệu trên slide phản ánh thời điểm tạo tài liệu; cần phân biệt PoPs, Regional Edge Caches và embedded PoPs khi so sánh.

#### KHÔNG NÊN làm
- **Trích 400+ Edge Locations/13 Regional Caches như số liệu hiện tại** → Vì trang AWS đã công bố phạm vi phủ sóng mới hơn; hãy nêu ngày kiểm tra và dẫn nguồn chính thức.

### AWS Free Tier

#### Giải thích cốt lõi
AWS thay đổi Free Tier cho khách hàng mới đăng ký từ **2025-07-15**. Account đăng ký từ ngày đó có lựa chọn Free Plan hoặc Paid Plan; Free Plan kết thúc sau 6 tháng hoặc khi hết credit, tùy điều kiện đến trước. Khách hàng mới nhận $100 credit và có thể nhận thêm tối đa $100 qua các hoạt động trên console. Tài khoản tạo trước ngày này tiếp tục thuộc chương trình legacy, gồm các ưu đãi có thời hạn 12 tháng nếu đáp ứng điều kiện. Vì vậy, dòng trên slide Trang 60 về 750 giờ Public IPv4/tháng trong 12 tháng không áp dụng chung cho mọi account mới. Trang VPC Pricing hiện liệt kê phí Public IPv4 là **$0.005/giờ cho mỗi địa chỉ** (in-use hoặc idle); hãy kiểm tra ưu đãi theo account và dịch vụ cụ thể. [Thông báo AWS Free Tier (2025-07-15)](https://aws.amazon.com/blogs/aws/aws-free-tier-update-new-customers-can-get-started-and-explore-aws-with-up-to-200-in-credits/) · [EC2 Free Tier theo ngày tạo account](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-free-tier-usage.html) · [Điều khoản Free Tier](https://aws.amazon.com/free/terms/) · [Định giá VPC - Public IPv4](https://aws.amazon.com/vpc/pricing/) (kiểm tra **2026-11-06**).

#### NÊN làm
- **Kiểm tra ngày tạo account, plan, credit và mức dùng đủ điều kiện** → Vì giới hạn khác nhau theo thời điểm đăng ký; theo dõi Billing/Free Tier để tránh dùng vượt ưu đãi.

#### KHÔNG NÊN làm
- **Giả định mọi account mới đều có 750 giờ Public IPv4 miễn phí trong 12 tháng** → Vì ưu đãi legacy phụ thuộc ngày tạo account; account mới theo chương trình 2025 dùng plan và credits có thời hạn/điều kiện khác.
