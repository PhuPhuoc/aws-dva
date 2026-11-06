# Section 5 - Amazon VPC (Trang 246-283)

---

## Phần 1: Nền tảng VPC (Trang 246-260)

### Tổng quan VPC (Trang 246-247)

**Sơ đồ các thành phần của VPC:**
```
Region
└─ VPC
   ├─ Internet Gateway
   ├─ Availability Zone
   │  ├─ Public subnet
   │  │  ├─ NACL
   │  │  ├─ Route Table → Internet Gateway
   │  │  ├─ NAT Gateway
   │  │  └─ Public EC2 instance + Security Group
   │  └─ Private subnet
   │     ├─ NACL
   │     ├─ Route Table → NAT Gateway / VPC Endpoint
   │     └─ Private EC2 instance + Security Group
   ├─ VPC Endpoint → S3 / DynamoDB
   ├─ VPC Peering ↔ VPC khác
   ├─ VPN Gateway ↔ Site-to-Site VPN ↔ Customer Gateway
   └─ Direct Connect ↔ DX Location
VPC Flow Logs → CloudWatch / S3 / DynamoDB
```

### Hiểu CIDR - IPv4: VPC và subnet (Trang 248)

- **CIDR = Classless Inter-Domain Routing** (định tuyến liên miền không phân lớp)
- Ví dụ: VPC `172.22.241.128/25` được chia thành public subnet `172.22.241.128/27` và private subnet `172.22.241.160/27` trong cùng một AZ

### Hiểu CIDR - Phần mạng và host (Trang 249)

- Với `172.22.241.128/25`, **25 bit** đầu xác định phần mạng; **7 bit** còn lại xác định host

### Hiểu CIDR - Dải địa chỉ (Trang 250)

- `172.22.241.128/25` bao phủ các địa chỉ từ `172.22.241.128` đến `172.22.241.255`, tổng cộng **128 địa chỉ IP**

### Hiểu CIDR - Subnet Mask (Trang 251)

| CIDR | Tổng số địa chỉ | Dải ví dụ bắt đầu tại `192.168.0.0` |
|---:|---:|---|
| `/32` | 1 | `192.168.0.0` |
| `/31` | 2 | `192.168.0.0`-`192.168.0.1` |
| `/30` | 4 | `192.168.0.0`-`192.168.0.3` |
| `/29` | 8 | `192.168.0.0`-`192.168.0.7` |
| `/28` | 16 | `192.168.0.0`-`192.168.0.15` |
| `/27` | 32 | `192.168.0.0`-`192.168.0.31` |
| `/26` | 64 | `192.168.0.0`-`192.168.0.63` |
| `/25` | 128 | `192.168.0.0`-`192.168.0.127` |
| `/24` | 256 | `192.168.0.0`-`192.168.0.255` |
| `/16` | 65.536 | `192.168.0.0`-`192.168.255.255` |
| `/0` | Tất cả địa chỉ IPv4 | `0.0.0.0`-`255.255.255.255` |

### Chia CIDR thành 4 subnet (Trang 252)

- Chia `172.22.241.128/25` thành bốn subnet `/27`:
  - `172.22.241.128/27`: `.128`-`.159`
  - `172.22.241.160/27`: `.160`-`.191`
  - `172.22.241.192/27`: `.192`-`.223`
  - `172.22.241.224/27`: `.224`-`.255`
- Mỗi subnet `/27` có **27 bit phần mạng** và **5 bit phần host**

### VPC mặc định (Trang 253)

- Mọi AWS account mới đều có **Default VPC**
- Nếu không chỉ định subnet, EC2 instance mới được khởi chạy trong Default VPC
- Default VPC có kết nối Internet; các instance trong đó có địa chỉ IPv4 public

### VPC trong AWS - IPv4 (Trang 254)

- **VPC = Virtual Private Cloud**
- Mỗi Region có thể có nhiều VPC (mặc định tối đa 5, giới hạn mềm); mỗi VPC có tối đa **5 CIDR blocks**
- Mỗi CIDR block có kích thước từ **/28** (16 địa chỉ) đến **/16** (65.536 địa chỉ)
- Nên chọn CIDR từ các dải IPv4 riêng theo RFC 1918: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`
- CIDR của VPC không được chồng lấn với các mạng khác (ví dụ mạng doanh nghiệp)

### Trạng thái ban đầu của VPC (Trang 255)

> *Sơ đồ: VPC đã được tạo trong Region nhưng chưa có subnet.*

### Thêm subnet vào VPC (Trang 256)

> *Sơ đồ: Thêm public và private subnet trong VPC, gắn với một Availability Zone.*


### VPC - Subnet (Trang 257)

**AWS giữ lại 5 địa chỉ IP** trong mỗi subnet:
1. Địa chỉ đầu tiên (ví dụ `10.0.0.0`) - địa chỉ mạng (**Network Address**)
2. Địa chỉ thứ hai (ví dụ `10.0.0.1`) - **VPC Router**
3. Địa chỉ thứ ba (ví dụ `10.0.0.2`) - ánh xạ đến DNS do Amazon cung cấp
4. Địa chỉ thứ tư (ví dụ `10.0.0.3`) - dành cho mục đích tương lai
5. Địa chỉ cuối cùng của subnet (ví dụ `10.0.0.255` trong `10.0.0.0/24`) - địa chỉ broadcast; AWS không hỗ trợ broadcast trong VPC nên địa chỉ này được giữ lại

**Mẹo làm bài:**
- Nếu cần **29 IP** cho EC2, không thể dùng `/27` (32 IP - 5 = 27)
- Cần chọn `/26` (64 IP - 5 = 59)

### Internet Gateway (IGW) (Trang 258-259)

- Cho phép resource trong VPC kết nối Internet
- Phải tạo IGW riêng với VPC
- Một VPC chỉ gắn được một IGW

**Lưu ý:**
- Chỉ có IGW thì chưa đủ để truy cập Internet
- Cần chỉnh Route Tables

### Chỉnh sửa Route Tables (Trang 260)

- Route Tables điều hướng traffic
- Public subnet cần có route đến IGW
- Private subnet **không có route trực tiếp** đến IGW

## Phần 2: VPC nâng cao (Trang 261-283)

### Bastion Host (Trang 261)

- **Bastion Host** dùng để SSH đến EC2 instances trong private subnets.
- Bastion nằm trong **public subnet** và kết nối đến các private subnets khác.
- **Security Group của Bastion:**
  - Cho phép inbound từ Internet qua port 22.
  - Giới hạn CIDR, ví dụ CIDR công khai của công ty.
- **Security Group của EC2 instances:**
  - Cho phép traffic từ Security Group của Bastion.
  - Hoặc cho phép traffic từ private IP của Bastion.

### NAT Gateway (Trang 262-263)

**Đặc điểm:**
- NAT do AWS quản lý.
- Băng thông cao và độ sẵn sàng cao.
- Không cần tự quản lý.
- Tính phí theo giờ sử dụng và lượng dữ liệu truyền.
- NAT Gateway được tạo trong một **AZ** cụ thể.
- Sử dụng **Elastic IP**.
- Cần có **Internet Gateway (IGW)**.

**Thông số kỹ thuật:**
- Băng thông **5 Gbps**, tự động mở rộng đến **100 Gbps**.
- Không có **Security Groups** để cấu hình.

### Network Access Control List (NACL) (Trang 264-266)

**Đặc điểm NACL:**
- Tường lửa kiểm soát lưu lượng đi vào và đi ra khỏi subnets.
- Mỗi subnet gắn với một NACL.
- Subnet mới được gán Default NACL.
- Có thể có quy tắc **ALLOW** và **DENY**.
- Quy tắc chỉ dựa trên địa chỉ IP.

**Default NACL:**
- Chấp nhận mọi lưu lượng inbound và outbound trong các subnet được gắn với nó.
- **Không sửa đổi Default NACL**; thay vào đó, tạo NACL tùy chỉnh.

### Security Group và NACL (Trang 267)

| Đặc điểm | Security Group | NACL |
|---------|---------------|------|
| **Cấp áp dụng** | Instance | Subnet |
| **Quy tắc** | Chỉ ALLOW | ALLOW và DENY |
| **Trạng thái** | **Stateful** | **Stateless** |
| **Đánh giá quy tắc** | Xét tất cả quy tắc | Theo số thứ tự tăng dần; quy tắc khớp đầu tiên quyết định kết quả |
| **Mặc định** | DENY inbound, ALLOW outbound | ALLOW tất cả |

### VPC Peering (Trang 268-269)

- Kết nối riêng tư hai VPCs qua mạng AWS.
- Hoạt động như thể hai VPCs thuộc cùng một mạng.
- **Yêu cầu:**
  - CIDRs không được chồng lấn.
  - Không có tính bắc cầu: A-B và B-C không tạo kết nối A-C.
  - Cập nhật route tables ở cả hai VPCs.

### VPC Endpoints (Trang 270-272)

**Hai loại:**

| Loại | Dịch vụ | Chi phí | Cách kết nối |
|------|---------|--------|--------------|
| **Gateway** | S3, DynamoDB | **Miễn phí** | Đích trong route table |
| **Interface** | Phần lớn dịch vụ AWS | Phí theo giờ và GB | ENI và Security Group |

**Lợi ích:**
- Truy cập riêng tư đến các dịch vụ AWS.
- Không cần IGW hoặc NAT Gateway.
- Được hỗ trợ bởi **AWS PrivateLink**.
- Có tính dự phòng và mở rộng theo chiều ngang.

### VPC Flow Logs (Trang 273-276)

**Thu thập thông tin:**
- Lưu lượng IP đi vào các giao diện mạng.
- Có thể bật ở cấp **VPC, Subnet hoặc ENI**.
- Đích lưu logs: **S3, CloudWatch Logs, Kinesis Data Firehose**.

**Các trường log:**
- `srcaddr`, `dstaddr`: địa chỉ IP nguồn và đích để nhận diện IP có vấn đề.
- `srcport`, `dstport`: port nguồn và đích để nhận diện port có vấn đề.
- `action`: **ACCEPT** hoặc **REJECT**.

### Site-to-Site VPN (Trang 277-278)

- Kết nối VPN tại chỗ đến AWS.
- Kết nối được mã hóa tự động.
- Lưu lượng đi qua Internet công cộng.

### Direct Connect (Trang 278)

- Thiết lập kết nối vật lý giữa môi trường tại chỗ và AWS.
- Kết nối riêng tư, bảo mật và tốc độ cao; lưu lượng đi qua mạng riêng.
- Thời gian thiết lập ít nhất một tháng.

### Site-to-Site VPN làm phương án dự phòng (Trang 279)

- Nếu Direct Connect bị gián đoạn, có thể dùng một kết nối Direct Connect dự phòng (chi phí cao) hoặc kết nối Site-to-Site VPN.

### Tóm tắt VPC (1/2) (Trang 280)

- **CIDR** là dải địa chỉ IP.
- **VPC** là Virtual Private Cloud.
- **Subnets** gắn với một **AZ**; ta xác định CIDR cho subnet.
- **Internet Gateway** ở cấp VPC, cung cấp truy cập Internet IPv4 và IPv6.
- Cần chỉnh **route tables** để thêm tuyến từ subnets đến IGW, VPC Peering, VPC Endpoints, v.v.
- **Bastion Host** là EC2 instance trong public subnet, dùng để SSH đến EC2 instances trong private subnets.
- **NAT Gateway** do AWS quản lý, cung cấp Internet có khả năng mở rộng cho EC2 instances trong private subnets khi đích là địa chỉ IPv4.

### Tóm tắt VPC (2/2) (Trang 281)

- **NACL** là stateless, áp dụng quy tắc inbound/outbound ở cấp subnet; cần tính đến **ephemeral ports**.
- **Security Groups** là stateful và hoạt động ở cấp EC2 instance.
- **VPC Peering** kết nối hai VPCs có CIDRs không chồng lấn và không có tính bắc cầu.
- **VPC Endpoints** cung cấp truy cập riêng tư đến các dịch vụ AWS như S3, DynamoDB, CloudFormation và SSM trong VPC.
- **VPC Flow Logs** có thể bật ở cấp VPC, subnet hoặc ENI; ghi lưu lượng ACCEPT/REJECT để nhận diện hoạt động đáng ngờ và phân tích bằng Athena hoặc CloudWatch Logs Insights.
- **Site-to-Site VPN** là VPN qua Internet công cộng giữa trung tâm dữ liệu tại chỗ và AWS.
- **Direct Connect** thiết lập kết nối riêng trực tiếp từ môi trường tại chỗ đến AWS.

### Định giá truyền dữ liệu S3 (Trang 282)

- Slide giới thiệu cách tính chi phí truyền dữ liệu thông thường.
- [Tài liệu Cloud Mentor Pro về chi phí truyền dữ liệu](https://blog.cloudmentor.pro/blog/aws/common-data-transfer-cost)

### Bài tập (Trang 283)

- Mạng - VPC

---


