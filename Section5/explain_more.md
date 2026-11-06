# Section 5 - Kiến thức mở rộng: Amazon VPC

---

## 1. Nền tảng Amazon VPC

### Lựa chọn kích thước VPC

#### Giải thích cốt lõi
Độ dài prefix CIDR quyết định tổng số địa chỉ trong VPC hoặc subnet: prefix càng ngắn thì dải địa chỉ càng lớn. AWS giữ lại 5 địa chỉ trong mỗi subnet, nên số IP khả dụng bằng tổng số địa chỉ trừ 5. Cần chọn kích thước đáp ứng số instance hiện tại và nhu cầu mở rộng.

| Kích thước VPC | Số IP khả dụng | Mục đích phổ biến |
|----------|------------|------------|
| /28 | 11 | Mạng nhỏ, cách ly |
| /27 | 27 | Một ứng dụng |
| /26 | 59 | Phòng ban nhỏ |
| /24 | 251 | Ứng dụng vừa |
| /16 | 65.531 | Mạng lớn, nhiều tầng |

**Các dải riêng RFC 1918:**
- `10.0.0.0/8` - dải lớn
- `172.16.0.0/12` - dải vừa; dải này chứa CIDR `172.31.0.0/16` của Default VPC. [Tài liệu AWS về thành phần Default VPC](https://docs.aws.amazon.com/vpc/latest/userguide/default-vpc-components.html).
- `192.168.0.0/16` - dải nhỏ

#### NÊN làm (DO)
- **Chọn CIDR đủ lớn cho các subnet và phần mở rộng dự kiến** → Tránh hết IP khi thêm tài nguyên. So với CIDR quá nhỏ, chọn dải phù hợp ngay từ đầu giảm nhu cầu đổi sơ đồ địa chỉ.
- **Dùng dải IPv4 riêng RFC 1918** → Các dải này dành cho mạng riêng và phù hợp với VPC hơn dải public IP.

#### KHÔNG NÊN làm (DON'T)
- **Dùng các CIDR bị chồng lấn** → Các mạng sẽ khó kết nối qua peering hoặc mạng doanh nghiệp. NÊN: lập kế hoạch các dải VPC/subnet không giao nhau.
- **Dùng dải địa chỉ IP công khai trong VPC hoặc tạo subnet quá nhỏ** → Có thể lãng phí/thiếu IP hoặc gây xung đột định tuyến. NÊN: dùng RFC 1918 và tính IP khả dụng sau khi trừ 5 địa chỉ AWS giữ lại.

### Thiết kế Subnet

#### Giải thích cốt lõi
Subnet chia VPC thành các vùng mạng có route table và security rules riêng. Public/private không do tên subnet quyết định mà phụ thuộc tuyến: subnet public có route ra Internet Gateway; workload cần cô lập hơn đặt trong private subnet. So với một subnet phẳng, phân tầng giúp tách luồng truy cập và triển khai qua nhiều AZ.

#### NÊN làm (DO)

```
VPC: 10.0.0.0/16
├── Public Subnet 1A (AZ 1): 10.0.1.0/24
│   └── NAT Gateway, Load Balancer
├── Private Subnet 1A (AZ 1): 10.0.2.0/24
│   └── Máy chủ ứng dụng
├── Private Subnet 2A (AZ 2): 10.0.3.0/24
│   └── Database
└── Dành riêng: 10.0.4.0/22 (mở rộng sau này)
```

**Thực hành tốt:**
- **Tách subnet public khỏi private** → Chỉ tài nguyên cần nhận kết nối Internet mới có route công khai; so với để mọi instance trong public subnet, cách này giảm bề mặt truy cập.
- **Phân bố subnet qua nhiều AZ** → Tránh phụ thuộc một AZ và tăng khả năng tiếp tục phục vụ khi AZ gặp sự cố, thay vì đặt toàn bộ workload ở một AZ.
- **Dùng `/24` làm điểm bắt đầu khi phù hợp** → Dễ có dư địa IP cho ứng dụng vừa; đây không phải kích thước bắt buộc, nên chọn nhỏ/lớn hơn theo số IP dự kiến.

#### KHÔNG NÊN làm (DON'T)
- **Đặt mọi workload vào một subnet duy nhất** → Khó tách quyền truy cập và tạo điểm lỗi tập trung. NÊN: tách public/private subnet theo vai trò và phân bố workload qua nhiều AZ.
- **Coi `/24` là kích thước bắt buộc cho mọi subnet** → CIDR quá lớn lãng phí dải địa chỉ, quá nhỏ có thể cạn IP. NÊN: ước tính số địa chỉ cần dùng và phần mở rộng trước khi chọn prefix.

### Internet Gateway và NAT Gateway

#### Giải thích cốt lõi
Internet Gateway cung cấp đường kết nối Internet IPv4/IPv6 ở cấp VPC; route table quyết định subnet nào dùng đường đó. NAT Gateway cho phép private instance khởi tạo lưu lượng IPv4 đi ra Internet qua IGW, nhưng không nhận kết nối mới từ Internet. So với NAT, IGW không che địa chỉ bằng NAT; cả hai đều cần route phù hợp.

**Internet Gateway (IGW):**
- Cho phép instance có IP public truy cập Internet.
- Hỗ trợ cả lưu lượng đi vào và đi ra.
- Dùng cho instance trong public subnet.

**NAT Gateway:**
- Cho phép instance có IP private truy cập Internet.
- Chỉ cho phép lưu lượng đi ra; không nhận lưu lượng đi vào từ Internet.
- Dùng cho instance trong private subnet và được đặt trong public subnet.
- Do AWS quản lý nên không cần tự quản lý hạ tầng.

#### NÊN làm (DO)

- Chọn IGW cho tài nguyên cần kết nối trực tiếp Internet; chọn NAT Gateway khi private instance chỉ cần mở kết nối IPv4 đi ra → Tránh mở đường inbound không cần thiết; so với NAT, IGW không thực hiện chuyển đổi địa chỉ.

**Khi nào dùng:**
| Nhu cầu | Giải pháp |
|---------|----------|
| Instance có IP public cần truy cập Internet | IGW |
| Instance trong private subnet cần truy cập Internet chiều đi ra | NAT Gateway |
| Không cần truy cập Internet | Không cần gateway |

#### KHÔNG NÊN làm (DON'T)
- **Cho rằng chỉ gắn IGW là subnet đã có Internet** → Không có route phù hợp thì lưu lượng không đi được; NÊN: thêm default route đến IGW cho public subnet.
- **Đặt NAT Gateway trong private subnet hoặc cho private route đi thẳng qua IGW** → NAT Gateway cần IGW ở public subnet làm đường ra. NÊN: đặt NAT Gateway trong public subnet và trỏ route IPv4 của private subnet đến NAT Gateway.

### Security Groups và NACLs

#### Giải thích cốt lõi
Security Group là firewall stateful ở cấp ENI/instance, còn NACL là firewall stateless ở cấp subnet. Security Group chỉ có rule ALLOW và tự cho phép traffic phản hồi; NACL có rule ALLOW/DENY theo thứ tự, nên phải cho phép tường minh cả chiều phản hồi. Với nhu cầu thông thường, Security Group là lớp kiểm soát chính; NACL bổ sung chặn ở ranh giới subnet.

**Security Groups (stateful):**
- Áp dụng ở cấp instance.
- **Stateful**: lưu lượng phản hồi được tự động cho phép.
- Mặc định: từ chối mọi lưu lượng inbound và cho phép mọi lưu lượng outbound.
- Xét tất cả các quy tắc.
- Có thể tham chiếu Security Groups khác.

**Network ACLs (stateless):**
- Áp dụng ở cấp subnet.
- **Stateless**: phải cho phép tường minh lưu lượng phản hồi ở chiều ngược lại.
- Mặc định: cho phép mọi lưu lượng inbound và outbound.
- Xét các quy tắc theo thứ tự số tăng dần.
- Không thể tham chiếu Security Groups khác.

#### NÊN làm (DO)

- Dùng Security Group làm lớp kiểm soát chính ở cấp instance; thêm NACL khi cần ALLOW/DENY ở cấp subnet → So với chỉ dùng NACL, Security Group cho phép tách rule theo workload và tự cho phép traffic phản hồi.

```
Kiến trúc:
Internet → IGW → Route Table → Public Subnet → NACL → Security Group → EC2
```

| Lớp | Chức năng | Loại |
|-------|----------|------|
| NACL | Lọc lưu lượng ở cấp subnet | Stateless (không lưu trạng thái) |
| Security Group | Lọc lưu lượng ở cấp instance | Stateful (có lưu trạng thái) |

#### KHÔNG NÊN làm (DON'T)
- **Dùng NACL thay Security Group để kiểm soát từng instance** → NACL tác động đến mọi tài nguyên trong subnet và khó biểu đạt rule theo workload. NÊN: dùng Security Group cho quyền ở cấp instance, thêm NACL khi cần chặn subnet-wide.
- **Quên rule phản hồi ở NACL stateless** → Kết nối có thể bị chặn dù chiều đi đã được cho phép. NÊN: khai báo cả hai chiều và các ephemeral ports phù hợp.

### VPC Endpoints

#### Giải thích cốt lõi
VPC Endpoint tạo đường riêng từ VPC đến dịch vụ AWS mà không cần route qua Internet công cộng. Gateway Endpoint thêm route trong route table cho S3/DynamoDB, không tạo ENI và không dùng AWS PrivateLink; Interface Endpoint dùng AWS PrivateLink, tạo ENI trong subnet và dùng Security Group. Gateway Endpoint không tính phí sử dụng endpoint, còn Interface Endpoint tính phí theo giờ và lượng dữ liệu xử lý. [Tài liệu AWS về VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/concepts.html).

#### NÊN làm (DO)

- Chọn Gateway Endpoint cho S3/DynamoDB; chọn Interface Endpoint cho dịch vụ hỗ trợ PrivateLink. So với NAT Gateway, endpoint giữ traffic dịch vụ trên đường riêng và Gateway Endpoint không tính phí sử dụng.

**Gateway Endpoints (S3, DynamoDB):**
- Không tính phí sử dụng endpoint.
- Dùng các tuyến trong Route Tables.
- Chỉ dùng cho S3 và DynamoDB.

```bash
# Tạo Gateway Endpoint cho S3
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-12345678 \
  --service-name com.amazonaws.us-east-1.s3 \
  --route-table-ids rtb-12345678
```

**Interface Endpoints:**
- Tính phí theo giờ và theo lượng dữ liệu xử lý.
- Tạo ENI (Elastic Network Interface - giao diện mạng ảo) trong subnet.
- Dùng cho các dịch vụ AWS khác ngoài S3 và DynamoDB.

**Ví dụ tạo Interface Endpoint cho Amazon EC2:**
```bash
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-12345678 \
  --service-name com.amazonaws.us-east-1.ec2 \
  --vpc-endpoint-type Interface \
  --subnet-ids subnet-12345678
```

#### KHÔNG NÊN làm (DON'T)
- **Cho rằng mọi VPC Endpoint đều dùng AWS PrivateLink hoặc đều miễn phí** → Có thể chọn sai loại endpoint, kết nối không hoạt động hoặc phát sinh chi phí ngoài dự kiến. Gateway Endpoint cho S3/DynamoDB dùng route table, không dùng PrivateLink và không tính phí sử dụng; Interface Endpoint dùng PrivateLink, tạo ENI, cần Security Group và tính phí theo giờ/GB. NÊN: phân biệt hai loại, kiểm tra dịch vụ hỗ trợ và cấu hình đúng Security Group.

### VPC Flow Logs

#### Giải thích cốt lõi
VPC Flow Logs ghi metadata về lưu lượng IP đi qua VPC, subnet hoặc ENI (ví dụ địa chỉ/cổng nguồn-đích, giao thức, `ACCEPT`/`REJECT`, số packets/bytes); đây không phải packet capture và không ghi payload. Có thể gửi logs đến CloudWatch Logs, S3 hoặc Kinesis Data Firehose; so với application logs, chúng tập trung vào metadata mạng thay vì hành vi bên trong ứng dụng.

#### NÊN làm (DO)

```bash
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-12345678 \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flow-logs \
  --deliver-logs-permission-arn arn:aws:iam::123456789012:role/VPCFlowLogsRole
```

- Cấp IAM role để Flow Logs ghi vào CloudWatch Logs; cấu hình policy/trust phù hợp với dịch vụ.
- Chọn `ACCEPT`, `REJECT` hoặc `ALL` theo nhu cầu phân tích.
- Bật Flow Logs ở cấp VPC, subnet hoặc ENI phù hợp với phạm vi cần chẩn đoán → So với bật ở mọi nơi, chọn phạm vi cần thiết giúp giới hạn lượng log và chi phí.

#### KHÔNG NÊN làm (DON'T)
- **Coi Flow Logs là packet capture hoặc kỳ vọng logs có payload ứng dụng** → Không thể dùng chúng để xem nội dung request hay chẩn đoán lỗi bên trong ứng dụng. NÊN: dùng application logs cho nội dung ứng dụng và Flow Logs cho metadata kết nối.
- **Bỏ qua quyền ghi của IAM role hoặc quyền truy cập tại đích logs** → Flow Logs có thể không được gửi đến nơi nhận. NÊN: cấu hình policy/trust và quyền tại đích trước khi phân tích.
- **Nhầm Flow Logs với S3 server access logs hoặc CloudTrail data events** → Các loại logs này ghi hoạt động khác nhau. NÊN: chọn nguồn log theo câu hỏi cần trả lời.

---

## 2. VPC Nâng cao - Bastion Host và NAT

### Thực hành tốt với Bastion Host

#### Giải thích cốt lõi
Bastion Host là máy trung gian trong public subnet, nhận SSH từ nguồn được giới hạn rồi chuyển tiếp đến EC2 private. So với mở SSH trực tiếp cho private instance, Bastion tạo một điểm kiểm soát tập trung; Systems Manager Session Manager là lựa chọn thay thế khi có thể quản trị instance qua SSM.

#### NÊN làm (DO)
- **Đặt Bastion trong public subnet** → Bastion cần nhận SSH từ nguồn quản trị rồi chuyển tiếp vào private subnet; so với đặt workload private trực tiếp ra Internet, chỉ một điểm trung gian được công khai.
- **Giới hạn SSH theo CIDR công ty và dùng Security Groups** → Thu hẹp nguồn truy cập so với mở port 22 toàn Internet.
- **Dùng SSH key được quản lý an toàn** → Tránh xác thực bằng mật khẩu dễ bị đoán; ưu tiên Session Manager khi có thể quản trị qua SSM mà không mở SSH.
- **Ưu tiên Systems Manager Session Manager khi phù hợp** → Giảm nhu cầu duy trì một máy Bastion so với truy cập SSH truyền thống.

#### KHÔNG NÊN làm (DON'T)
- **Cho phép SSH từ `0.0.0.0/0`** → Bastion có thể bị dò quét và tấn công từ toàn Internet. NÊN: giới hạn theo CIDR công ty hoặc dùng Session Manager.
- **Dùng mật khẩu hoặc lưu private key trên Bastion** → Lộ instance có thể làm lộ thông tin xác thực. NÊN: dùng SSH key được quản lý an toàn hoặc Session Manager.
- **Dùng Bastion cho mục đích khác ngoài quản trị** → Tăng bề mặt tấn công và khó kiểm soát quyền. NÊN: dành riêng máy cho truy cập quản trị.

**Ví dụ: dùng Session Manager thay cho Bastion**
```bash
aws ssm start-session --target <instance-id>
```

### NAT Gateway và NAT Instance

#### Giải thích cốt lõi
NAT Gateway là dịch vụ được AWS quản lý, mở rộng tự động và không có Security Group; NAT Instance là EC2 do người dùng vận hành, cho phép tùy chỉnh sâu hơn nhưng cần tự quản lý capacity, bản vá và tính sẵn sàng. Chọn NAT Gateway cho lưu lượng private subnet trong môi trường production; chỉ chọn NAT Instance khi cần khả năng tùy biến riêng.

#### NÊN làm (DO)

- Ưu tiên NAT Gateway khi cần dịch vụ được AWS quản lý và độ sẵn sàng cao; chỉ dùng NAT Instance khi cần tùy chỉnh như port forwarding hoặc kiểm soát hệ điều hành.

| Tình huống | Khuyến nghị |
|------------|-------------|
| Cần truy cập Internet chiều đi ra cho instances trong private subnet | **NAT Gateway** |
| Cần chuyển tiếp port đến private instance | **NAT Instance** |
| Lưu lượng nhỏ, ưu tiên tiết kiệm | NAT Instance (ví dụ `t2.micro`) |
| Môi trường production cần độ sẵn sàng cao | **NAT Gateway** |

**Ưu điểm của NAT Gateway:**
- Do AWS quản lý, có độ sẵn sàng cao.
- Băng thông tự động mở rộng từ 5 đến 100 Gbps.
- Không cần tự bảo trì.
- Hỗ trợ IPv4; với IPv6 dùng Egress-only Internet Gateway.

**Lưu ý:** NAT Gateway được tạo trong một AZ. Để tăng độ sẵn sàng, tạo NAT Gateway ở nhiều AZs và cấu hình tuyến tương ứng.

#### KHÔNG NÊN làm (DON'T)
- **Dùng một NAT Gateway cho workload nhiều AZ mà không chấp nhận phụ thuộc một AZ** → Sự cố AZ có thể làm mất đường ra hoặc buộc lưu lượng liên AZ. NÊN: tạo NAT Gateway theo AZ và route subnet đến NAT Gateway cùng AZ.
- **Dùng NAT Instance trong production chỉ để giảm chi phí mà không vận hành HA/bản vá** → Instance lỗi hoặc quá tải làm gián đoạn outbound. NÊN: dùng NAT Gateway nếu ưu tiên độ sẵn sàng và vận hành đơn giản.

---

## 3. Bảo mật VPC - NACL và Security Groups

### Khi nào dùng NACL

#### Giải thích cốt lõi
NACL hữu ích khi cần ALLOW hoặc DENY địa chỉ IP ở cấp subnet; Security Group thường đủ cho kiểm soát kết nối ở cấp instance. NACL áp dụng cho mọi resource trong subnet và xử lý rule theo số thứ tự, nên phù hợp để chặn mạng/IP diện rộng hơn là thay thế chính sách instance.

#### NÊN làm (DO)
- **Chặn IP độc hại bằng rule DENY ở cấp subnet** → Không phải lặp lại rule ở từng Security Group; phù hợp hơn khi cần chặn cùng IP trên nhiều instance.
- **Tách NACL cho subnet có yêu cầu khác nhau** → Tránh áp dụng chung một bộ rule cho mọi tầng mạng.
- **Khai báo ephemeral ports cho traffic phản hồi** → Vì NACL stateless, thiếu rule này có thể làm kết nối bị chặn dù chiều đi được cho phép.

#### KHÔNG NÊN làm (DON'T)
- **Sửa Default NACL trực tiếp** → Có thể làm thay đổi lưu lượng của mọi subnet đang dùng NACL đó. NÊN: tạo NACL tùy chỉnh rồi gắn riêng cho subnet cần chính sách khác.
- **Chỉ dựa vào NACL khi Security Group đã đáp ứng yêu cầu** → Thêm rule trùng lặp làm khó gỡ lỗi. NÊN: dùng NACL khi thực sự cần ALLOW/DENY ở cấp subnet.
- **Dùng NACL để kiểm soát bảo mật ở cấp ứng dụng** → NACL không hiểu danh tính hay logic ứng dụng. NÊN: dùng Security Group và cơ chế kiểm soát ở tầng ứng dụng.

### Cổng ephemeral trong NACL

#### Giải thích cốt lõi
Vì NACL stateless, response traffic không tự được cho phép. Client thường chọn ephemeral port cho đầu cuối của kết nối; NACL phải cho phép dải cổng phản hồi thích hợp ở chiều ngược lại. Security Group stateful không cần rule phản hồi riêng, nhưng NACL thì cần.

#### NÊN làm (DO)
- Cho phép các ephemeral ports phù hợp cho lưu lượng phản hồi; vì NACL stateless nên cần có quy tắc cho cả hai chiều.

**Các dải ephemeral port thường gặp:**
- Linux: 32768-60999.
- Windows: 49152-65535.
- AWS khuyến nghị: **1024-65535**.

**Thứ tự đánh giá quy tắc NACL:**
NACL xét quy tắc theo số thứ tự tăng dần và dừng tại quy tắc khớp đầu tiên. Đặt rule DENY có số nhỏ hơn các rule ALLOW rộng hơn nếu cần chặn IP đó trên mọi port.
```
Rule #: 100 - Deny IP độc hại trên mọi traffic
Rule #: 110 - Allow HTTP (80)
Rule #: 120 - Allow HTTPS (443)
Rule #: *   - Deny traffic còn lại
```

#### KHÔNG NÊN làm (DON'T)
- **Mở toàn bộ ephemeral ports mà không xét yêu cầu và rule chiều đi** → Làm rộng phạm vi traffic được phép hơn mức cần thiết. NÊN: dùng dải phù hợp hệ điều hành/client và kiểm tra cả hai chiều; AWS khuyến nghị dải `1024-65535` khi cần phạm vi rộng.

---

## 4. VPC Peering

### Trường hợp sử dụng VPC Peering

#### Giải thích cốt lõi
VPC Peering tạo kết nối riêng tư giữa hai VPC có CIDR không chồng lấn; cần route ở cả hai phía và kết nối không bắc cầu. So với VPC Peering, Transit Gateway phù hợp hơn khi cần kết nối nhiều VPC theo mô hình hub-and-spoke vì quản lý kết nối tập trung.

#### NÊN làm (DO)
```
VPC-A (10.0.0.0/16) ←→ VPC-B (10.1.0.0/16)
```
- **Dùng peering cho các cặp VPC cần kết nối trực tiếp** → Tạo đường riêng giữa hai mạng; so với kết nối qua Internet, traffic giữa hai VPC đi trên mạng AWS.
- **Cập nhật route table ở cả hai phía và tránh CIDR chồng lấn** → Bảo đảm địa chỉ đích có route và không bị mơ hồ.
- **Dùng Transit Gateway thay vì nhiều peering khi cần kết nối tập trung nhiều VPC** → Giảm số kết nối phải quản lý so với mô hình full-mesh.

**LƯU Ý QUAN TRỌNG:**
- Peering không bắc cầu: A-B không đồng nghĩa A có thể giao tiếp với C.
- Cập nhật bảng định tuyến (route tables) ở cả hai VPCs.
- CIDRs không được chồng lấn.

#### KHÔNG NÊN làm (DON'T)
- **Giả định peering bắc cầu hoặc chỉ thêm route một phía** → Các VPC không liên lạc được như mong muốn. NÊN: tạo peering trực tiếp cho từng cặp cần kết nối và cập nhật route tables ở cả hai VPC.
- **Dùng peering cho mạng hub-and-spoke lớn mà không tính số kết nối** → Số quan hệ peering tăng và khó vận hành. NÊN: cân nhắc Transit Gateway khi cần kết nối tập trung.

## 5. Trọng tâm ôn tập VPC

- So sánh **Security Groups** (stateful, cấp instance) với **NACLs** (stateless, cấp subnet; rule khớp đầu tiên có hiệu lực).
- Nhớ cập nhật route tables, CIDRs không chồng lấn và tính không bắc cầu của VPC Peering.
- Chọn Gateway Endpoint cho S3/DynamoDB; chọn Interface Endpoint cho phần lớn dịch vụ AWS khác.
- Kiểm tra VPC Flow Logs ở cấp VPC, subnet hoặc ENI; phân biệt ACCEPT và REJECT.
- Khi dùng NACL, cho phép lưu lượng phản hồi qua ephemeral ports.


### Kiến trúc VPC nhiều tầng

#### Giải thích cốt lõi
Sơ đồ phân luồng từ Internet Gateway đến public subnet, nơi đặt điểm nhận lưu lượng/NAT Gateway, rồi đến workload trong private subnet. VPC Endpoint tạo đường riêng đến S3/DynamoDB. So với đặt mọi tài nguyên ở public subnet, cách phân tầng giới hạn đường truy cập trực tiếp.

#### NÊN làm (DO)
- Đặt các điểm cần nhận lưu lượng Internet trong public subnet, còn application/database trong private subnet → Giảm bề mặt truy cập; so với mạng phẳng, route và Security Group giới hạn luồng giữa các tầng.

```
Internet
    ↓
Internet Gateway
    ↓
Route Table
    ↓
┌─────────────────────────────────────┐
│ Public Subnet (AZ1)                 │
│ - NAT Gateway                       │
│ - Application Load Balancer          │
├─────────────────────────────────────┤
│ Private Subnet (AZ1)                │
│ - EC2 Instances (ứng dụng)          │
│ - Auto Scaling Group                 │
├─────────────────────────────────────┤
│ Private Subnet (AZ2)                │
│ - RDS Multi-AZ                       │
│ - ElastiCache                        │
└─────────────────────────────────────┘
    ↓
VPC Endpoint (S3, DynamoDB)
```

#### KHÔNG NÊN làm (DON'T)
- **Đặt database hoặc application instance có thể truy cập trực tiếp từ Internet** → Tăng nguy cơ truy cập trái phép. NÊN: giữ workload trong private subnet, chỉ công khai điểm nhận traffic cần thiết và dùng VPC Endpoint cho dịch vụ AWS phù hợp.

---
### Tra cứu nhanh CIDR cho VPC

| CIDR | Tổng số IP | IP khả dụng |
|------|-----------|------------|
| /28 | 16 | 11 |
| /27 | 32 | 27 |
| /26 | 64 | 59 |
| /25 | 128 | 123 |
| /24 | 256 | 251 |
| /23 | 512 | 507 |
| /22 | 1.024 | 1.019 |
| /21 | 2.048 | 2.043 |
| /20 | 4.096 | 4.091 |
| /19 | 8.192 | 8.187 |
| /18 | 16.384 | 16.379 |
| /17 | 32.768 | 32.763 |
| /16 | 65.536 | 65.531 |
---


