# Section 5 - Câu hỏi ôn tập DVA: Amazon VPC
---

## 1. VPC Nâng cao

### Question 1:
A company needs to SSH into private EC2 instances from the Internet. What should be implemented?

```
A. NAT Gateway
B. Internet Gateway
C. Bastion Host
D. VPC Peering
```

### ✅ Answer: **C. Bastion Host**

### 📖 Explanation (Vietnamese):

**Tại sao chọn C:**
- **Bastion Host** là EC2 instance trong public subnet, dùng làm máy trung gian để SSH đến các EC2 instances trong private subnets.
- Luồng kết nối: Internet → Bastion Host → EC2 instance riêng tư.
- Security Group của Bastion chỉ nên cho phép SSH từ CIDR cần thiết; Security Group của EC2 riêng tư cho phép traffic từ Security Group của Bastion.

**Kiến trúc:**
```
Internet → Bastion Host (Public) → Private EC2 Instances
```

**Tại sao loại các lựa chọn khác:**

| Lựa chọn | Lý do loại trừ |
|----------|----------------|
| A. NAT Gateway | Chỉ cung cấp kết nối outbound, không cho phép SSH vào private instances |
| B. Internet Gateway | Không tự tạo đường SSH đến EC2 riêng tư; cần Bastion hoặc cơ chế quản trị khác |
| D. VPC Peering | Kết nối hai VPCs, không phải giải pháp SSH vào private instance |

### 📝 Phân tích thuật ngữ:
- **Bastion Host**: máy trung gian trong public subnet để truy cập mạng riêng.
- **Jump server**: máy chủ trung gian để truy cập mạng riêng.

---

### Question 2:
What is the main difference between Security Groups and NACLs?

```
A. Security Groups are stateless, NACLs are stateful
B. Security Groups are stateful, NACLs are stateless
C. Both are stateful
D. Both are stateless
```

### ✅ Answer: **B. Security Groups are stateful, NACLs are stateless**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**

| Đặc điểm | Security Group | NACL |
|----------|----------------|------|
| **Cấp áp dụng** | Instance | Subnet |
| **Trạng thái** | **Stateful** | **Stateless** |
| **Rules** | Chỉ ALLOW | ALLOW và DENY |
| **Đánh giá** | Xét tất cả rules | Theo thứ tự tăng dần; rule khớp đầu tiên quyết định kết quả |
| **Traffic phản hồi** | Tự động được cho phép | Cần rule cho cả hai chiều |

- **Stateful:** khi cho phép inbound, traffic phản hồi tự động được phép.
- **Stateless:** cần cho phép riêng traffic phản hồi ở chiều ngược lại.

**Tại sao loại các lựa chọn khác:**
- A đảo ngược đặc điểm: Security Groups stateful, NACLs stateless.
- C và D đều sai vì một loại stateful, loại còn lại stateless.

---

### Question 3:
A VPC has two private subnets in different AZs. Instances need to access S3 without going over the Internet. What is the best solution?

```
A. NAT Gateway
B. Internet Gateway
C. VPC Endpoint (Gateway type)
D. VPC Peering
```

### ✅ Answer: **C. VPC Endpoint (Gateway type)**

### 📖 Explanation (Vietnamese):

**Tại sao chọn C:**
- **VPC Gateway Endpoint** cho phép truy cập riêng tư đến S3/DynamoDB mà không đi qua Internet.
- Gateway Endpoint không tính phí sử dụng endpoint và dùng route table để định tuyến.

**So sánh hai loại VPC Endpoints:**

| Loại | Services | Chi phí | Cách kết nối |
|------|----------|--------|--------------|
| **Gateway** | S3, DynamoDB | Miễn phí | Route table |
| **Interface** | Các dịch vụ khác | Phí theo giờ và GB | ENI và Security Group |

**Tại sao loại các lựa chọn khác:**
- A. NAT Gateway cung cấp outbound Internet, không phải private access đến S3.
- B. Internet Gateway đưa traffic ra Internet công cộng.
- D. VPC Peering kết nối VPCs, không phải AWS services.

---

### Question 4:
A company wants to block traffic from a specific malicious IP address at the subnet level. Which AWS feature should be used?

```
A. Security Group
B. NACL with DENY rule
C. VPC Peering
D. Route Table
```

### ✅ Answer: **B. NACL with DENY rule**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- **NACLs** hỗ trợ rule **DENY** và áp dụng ở cấp subnet.
- Rules được xét theo số thứ tự tăng dần; việc xét dừng tại rule khớp đầu tiên.
- Đặt rule chặn trước các rule ALLOW rộng hơn để chặn IP độc hại trên mọi port cần thiết.

**Ví dụ NACL:**
```
Rule #: 100 - DENY IP độc hại trên mọi traffic
Rule #: 110 - ALLOW HTTP (80)
Rule #: 120 - ALLOW HTTPS (443)
Rule #: *   - DENY traffic còn lại
```

**Tại sao loại các lựa chọn khác:**

| Lựa chọn | Lý do loại trừ |
|----------|----------------|
| A. Security Group | Chỉ hỗ trợ ALLOW rules, không có DENY |
| C. VPC Peering | Kết nối VPCs, không chặn IP |
| D. Route Table | Định tuyến traffic, không phải tường lửa chặn IP |

---

### Question 5:
What is the bandwidth capacity of a NAT Gateway?

```
A. 1 Gbps
B. 5 Gbps (auto-scaling to 45 Gbps)
C. 5 Gbps (auto-scaling to 100 Gbps)
D. 10 Gbps
```

### ✅ Answer: **C. 5 Gbps (auto-scaling to 100 Gbps)**

### 📖 Explanation (Vietnamese):

**Tại sao chọn C:**
- **NAT Gateway** bắt đầu ở mức **5 Gbps** và tự động mở rộng đến **100 Gbps**.
- Do AWS quản lý nên không cần cấu hình Security Groups; tính phí theo giờ và lượng dữ liệu truyền.

**Tại sao loại các lựa chọn khác:**

| Lựa chọn | Lý do loại trừ |
|----------|----------------|
| A | Thấp hơn băng thông ban đầu của NAT Gateway |
| B | Băng thông tối đa 45 Gbps không đúng; mức tối đa trong đáp án đúng là 100 Gbps |
| D | 10 Gbps không phải giới hạn mở rộng tối đa được nêu trong slide |

---

## 2. CIDR và NAT Gateway

### Question 6:
A company uses the subnet `10.0.0.0/28`. How many IPv4 addresses can it assign to EC2 instances?

```
A. 16
B. 14
C. 11
D. 8
```

### ✅ Answer: **C. 11**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- Subnet `/28` có tổng cộng **16 địa chỉ IP**.
- AWS **giữ lại 5 địa chỉ IP** trong mỗi subnet:
  1. `10.0.0.0` - địa chỉ mạng (Network Address)
  2. `10.0.0.1` - VPC Router
  3. `10.0.0.2` - Amazon DNS
  4. `10.0.0.3` - địa chỉ dành cho mục đích tương lai
  5. `10.0.0.15` - địa chỉ cuối cùng của subnet `/28`, được AWS giữ lại
- **Số IP khả dụng** = 16 - 5 = **11**

**Tham khảo nhanh:**
| CIDR | Tổng số IP | IP khả dụng |
|------|-----------|------------|
| /28 | 16 | **11** |
| /27 | 32 | 27 |
| /26 | 64 | 59 |
| /24 | 256 | 251 |

**Mẹo làm bài:**
- Cần 29 IP → Không thể dùng `/27` (32 - 5 = 27 < 29).
- Cần dùng `/26` (64 - 5 = 59 > 29).

### Question 7:
A private EC2 instance needs to **download software from the Internet** but should not be directly accessible from the Internet. What is needed?

```
A. Internet Gateway
B. NAT Gateway in public subnet
C. VPC Peering
D. Direct Connect
```

### ✅ Answer: **B. NAT Gateway in public subnet**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **NAT Gateway** cho phép:
  - EC2 trong private subnet **chủ động khởi tạo** lưu lượng đi ra Internet.
  - Lưu lượng phản hồi cho kết nối đã khởi tạo đi ngược về qua NAT.
  - Không cho phép kết nối mới được khởi tạo từ Internet đi vào private instance.

**Sơ đồ luồng:**
```
Private EC2 → Private Subnet Route → NAT Gateway → IGW → Internet
```

**Vị trí NAT Gateway:**
- NAT Gateway phải nằm trong **public subnet**.
- Public subnet phải có route đến **Internet Gateway**.

**Why other options were eliminated:**

| Lựa chọn | Lý do loại trừ |
|--------|----------------|
| A. IGW | Chỉ cho phép instance có public IP kết nối Internet, không thực hiện NAT. |
| C. VPC Peering | Kết nối các VPC, không cung cấp truy cập Internet. |
| D. Direct Connect | Kết nối tại chỗ với AWS, không thực hiện NAT cho instance. |

### 📝 Vocabulary Analysis:
- **NAT Gateway**: Network Address Translation Gateway
- **Outbound traffic**: Traffic từ private network ra Internet
- **Public subnet**: Subnet có route đến IGW

---



## Bảng tóm tắt

| Chủ đề | Điểm chính |
|--------|------------|
| **Bastion Host** | Máy trung gian trong public subnet để SSH đến EC2 instances riêng tư |
| **CIDR/Subnet** | /28 có 11 IP khả dụng; AWS giữ lại 5 địa chỉ trong mỗi subnet |
| **NACL** | Stateless, cấp subnet, hỗ trợ DENY; rule khớp đầu tiên quyết định kết quả |
| **Security Groups** | Stateful, cấp instance, chỉ có ALLOW |
| **VPC Endpoint** | Gateway Endpoint cho phép truy cập riêng tư đến S3/DynamoDB |
| **NAT Gateway** | Managed outbound Internet access cho workload IPv4 trong private subnet |
| **VPC Peering** | Kết nối riêng, không bắc cầu, CIDRs không chồng lấn |
| **VPC Flow Logs** | Ghi traffic ACCEPT/REJECT ở cấp VPC, subnet hoặc ENI |
