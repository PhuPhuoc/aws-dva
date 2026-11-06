# Section 3 - RDS, Aurora & ElastiCache | Route 53


## Phần 1: RDS, Aurora & ElastiCache (Trang 131-156)

### Tổng quan Amazon RDS (Trang 132)

- **RDS** là viết tắt của Relational Database Service
- Dịch vụ database được quản lý, sử dụng **SQL** làm ngôn ngữ truy vấn
- Cho phép tạo database trên cloud để AWS quản lý
- Hỗ trợ: PostgreSQL, MySQL, MariaDB, Oracle, Microsoft SQL Server, IBM DB2 và Aurora

### Ưu điểm của RDS so với việc triển khai database trên EC2 (Trang 133)

- Cấp phát tự động và vá hệ điều hành
- Sao lưu liên tục và khôi phục về một thời điểm cụ thể (**Point-in-Time Restore**)
- Bảng điều khiển giám sát
- **Read Replicas** cải thiện hiệu năng đọc
- Cấu hình **Multi-AZ** hỗ trợ khôi phục sau thảm họa (**Disaster Recovery**)
- Khung thời gian bảo trì cho các lần nâng cấp
- Có thể mở rộng theo chiều dọc và chiều ngang
- Storage được lưu trên EBS (gp2 hoặc io1)

### RDS - Tự động mở rộng dung lượng lưu trữ (Trang 134)

- Tự động tăng dung lượng lưu trữ cho RDS DB instance khi cần
- Khi RDS phát hiện sắp hết dung lượng database còn trống, dịch vụ sẽ tự động mở rộng storage
- Cần đặt ngưỡng storage tối đa (**Maximum Storage Threshold**)
- RDS tự điều chỉnh storage khi:
  - Dung lượng trống dưới 10% dung lượng đã cấp phát
  - Tình trạng thiếu dung lượng kéo dài ít nhất 5 phút
  - Đã qua 6 giờ kể từ lần điều chỉnh gần nhất
- Hữu ích cho workload khó dự đoán
- Hỗ trợ mọi RDS database engine

### RDS Read Replicas để mở rộng khả năng đọc (Trang 135-137)

- Có thể tạo tối đa **15 Read Replicas**
- Có thể đặt trong cùng AZ, khác AZ hoặc khác Region
- Replication là **ASYNC**, vì vậy dữ liệu ở replica có thể bị trễ và cuối cùng mới nhất quán (**eventually consistent**)
- Có thể promote replica thành database độc lập
- Ứng dụng cần cập nhật connection string để sử dụng Read Replicas

**Trường hợp sử dụng:**
- Database production đang xử lý tải thông thường
- Tạo Read Replica để chạy ứng dụng báo cáo hoặc truy vấn phân tích
- Workload của ứng dụng production không bị ảnh hưởng
- Dùng Read Replicas cho câu lệnh đọc (`SELECT`), không dùng cho `INSERT`, `UPDATE` hay `DELETE`

**Lưu ý:** Trong cùng một Region, không tính phí truyền dữ liệu cross-AZ cho RDS Read Replicas.

### RDS Multi-AZ (Khôi phục sau thảm họa) (Trang 138)

- Replication **SYNC**
- Có một DNS endpoint; RDS tự động failover sang standby và cập nhật endpoint để ứng dụng reconnect
- Tăng tính sẵn sàng (**availability**)
- Failover có thể xảy ra khi mất AZ hoặc mạng, hay khi instance hoặc storage gặp sự cố
- Ứng dụng không cần thay đổi thủ công
- Không dùng Multi-AZ để mở rộng khả năng xử lý workload
- Read Replicas cũng có thể được cấu hình **Multi-AZ** để phục vụ khôi phục sau thảm họa (DR).

### Chuyển từ Single-AZ sang Multi-AZ (Trang 139)

1. Chọn **Modify** cho database
2. Tạo snapshot
3. Khôi phục database mới từ snapshot tại AZ khác
4. Thiết lập đồng bộ hóa giữa hai database

### Amazon Aurora (Trang 140)

- Aurora là công nghệ độc quyền của AWS, không phải phần mềm mã nguồn mở
- Aurora hỗ trợ PostgreSQL và MySQL
- AWS mô tả Aurora là dịch vụ được tối ưu cho AWS Cloud: hiệu năng được nêu là cao gấp 5 lần MySQL trên RDS và hơn 3 lần PostgreSQL trên RDS
- Storage của Aurora tự tăng theo từng bước 10 GB, tối đa **128 TB**
- Aurora hỗ trợ tối đa **15 replicas**; replication nhanh hơn MySQL (replica lag dưới 10 ms)
- Failover gần như tức thời; khả năng sẵn sàng cao được tích hợp sẵn
- Aurora có chi phí cao hơn RDS (20%) nhưng hiệu quả hơn

### Aurora: Tính sẵn sàng cao và mở rộng khả năng đọc (Trang 141)

- Có **6 bản sao** dữ liệu trên **3 AZ**:
  - Cần 4/6 bản sao để xử lý thao tác ghi
  - Cần 3/6 bản sao để xử lý thao tác đọc
- Tự sửa chữa bằng replication ngang hàng
- Dữ liệu được phân bố trên hàng trăm volume

**Kiến trúc:**
- Một Aurora instance làm primary để xử lý thao tác ghi
- Tự động failover primary trong vòng dưới 30 giây
- Primary và tối đa 15 Aurora Read Replicas phục vụ thao tác đọc
- Hỗ trợ replication cross-Region

### Aurora DB Cluster: Endpoints (Trang 142)

> *Sơ đồ: Writer Endpoint trỏ đến primary để xử lý thao tác ghi; Reader Endpoint cân bằng kết nối đọc giữa các Aurora Replicas. Các instance dùng chung storage volume tự tăng từ 10 GB đến 128 TB.*

### Các tính năng của Aurora (Trang 143)

- Tự động failover
- Sao lưu và khôi phục
- Cách ly và bảo mật
- Tuân thủ tiêu chuẩn ngành
- Mở rộng chỉ bằng thao tác
- Vá tự động, không downtime
- Giám sát nâng cao
- Bảo trì định kỳ
- **Backtrack**: khôi phục dữ liệu về một thời điểm trước đó mà không cần dùng backup

### Bảo mật RDS và Aurora (Trang 144)

**Mã hóa khi lưu trữ:**
- Database primary và replicas dùng AWS KMS; cần cấu hình lúc khởi tạo
- Nếu primary không được mã hóa, không thể mã hóa các Read Replicas
- Để mã hóa database chưa mã hóa: tạo snapshot rồi khôi phục thành database đã mã hóa

**Mã hóa khi truyền:**
- TLS sẵn sàng sử dụng theo mặc định
- Dùng AWS TLS root certificates ở phía client

**Xác thực và truy cập:**
- **IAM Authentication**: dùng IAM roles để kết nối thay vì username/password
- **Security Groups** kiểm soát quyền truy cập mạng
- Không hỗ trợ SSH, ngoại trừ RDS Custom
- Có thể bật Audit Logs và gửi đến CloudWatch Logs

### Amazon RDS Proxy (Trang 145)

- Database proxy được quản lý toàn phần cho RDS
- Ứng dụng có thể gom nhóm và chia sẻ các kết nối database
- Cải thiện hiệu quả database bằng cách giảm áp lực lên CPU/RAM, giảm số kết nối mở và hạn chế timeout
- Không cần quản lý máy chủ, tự mở rộng và có tính sẵn sàng cao (Multi-AZ)
- Có thể giảm thời gian failover của RDS và Aurora tới 66%
- Hỗ trợ RDS (MySQL, PostgreSQL, MariaDB, MS SQL Server) và Aurora (MySQL, PostgreSQL)
- Hầu hết ứng dụng không cần thay đổi code
- Có thể bắt buộc dùng **IAM Authentication** cho database và lưu credentials an toàn trong **AWS Secrets Manager**
- RDS Proxy không public; phải truy cập từ VPC

### Tổng quan Amazon ElastiCache (Trang 146)

- Tương tự RDS cung cấp database quan hệ được quản lý, **ElastiCache** cung cấp Redis hoặc Memcached được quản lý
- Cache là database trong bộ nhớ, có hiệu năng cao và độ trễ thấp
- Giảm tải cho database trong workload đọc nhiều
- Giúp ứng dụng không lưu trạng thái cục bộ (**stateless**)
- AWS quản lý bảo trì/vá hệ điều hành, tối ưu hóa, cài đặt, cấu hình, giám sát, khôi phục sự cố và backup
- Sử dụng ElastiCache thường cần thay đổi đáng kể trong code ứng dụng

### ElastiCache: Kiến trúc giải pháp - DB Cache (Trang 147)

**1. Mẫu DB Cache:**
- Ứng dụng truy vấn ElastiCache; nếu cache miss thì lấy dữ liệu từ RDS rồi lưu vào ElastiCache
- Giúp giảm tải cho RDS
- Cần có chiến lược vô hiệu hóa cache để bảo đảm dữ liệu dùng là mới nhất

### ElastiCache: Kiến trúc giải pháp - User Session Store (Trang 148)

- Người dùng đăng nhập vào bất kỳ application instance nào
- Ứng dụng ghi dữ liệu session vào ElastiCache
- Khi người dùng chuyển sang instance khác, instance đó đọc session từ ElastiCache và người dùng vẫn giữ trạng thái đăng nhập

### ElastiCache - Redis và Memcached (Trang 149)

| Đặc điểm | **Redis** | **Memcached** |
|---------|-----------|---------------|
| **Replication** | Multi-AZ, tự động failover | Sharding (nhiều node) |
| **Tính sẵn sàng cao** | Có (Read Replicas) | Không |
| **Độ bền dữ liệu** | AOF persistence | Không lưu bền |
| **Backup và khôi phục** | Có | Không |
| **Kiểu dữ liệu** | Sets, Sorted Sets | Key-value đơn giản |
| **Kiến trúc** | Replication | Sharding |
| **Mô hình luồng** | Đơn luồng | Đa luồng |

### Những điểm cần cân nhắc khi triển khai caching (Trang 150)

**Cần đặt ra các câu hỏi:**
- Có an toàn khi cache dữ liệu không? Dữ liệu có thể cũ và chỉ nhất quán cuối cùng
- Cache có hiệu quả không? Cache phù hợp khi dữ liệu thay đổi chậm và một số key được yêu cầu thường xuyên
- Dữ liệu có cấu trúc phù hợp không? Ví dụ: key-value hoặc kết quả tổng hợp

**Các anti-pattern:**
- Dùng với dữ liệu thay đổi nhanh
- Dùng khi toàn bộ keyspace lớn đều được yêu cầu thường xuyên

### Lazy Loading / Cache-Aside (Trang 151-152)

> *Sơ đồ: Luồng xử lý theo mẫu Cache-Aside*

**Ưu điểm:**
- Chỉ cache dữ liệu được yêu cầu; không lấp cache bằng dữ liệu không sử dụng
- Node gặp sự cố không làm hệ thống ngừng hoạt động; chỉ tăng độ trễ trong lúc làm nóng cache

**Nhược điểm:**
- Cache miss làm tăng độ trễ vì cần 3 lượt trao đổi
- Dữ liệu cũ: database có thể đã được cập nhật trong khi cache vẫn chứa dữ liệu cũ

### Write-Through (Trang 153-154)

- Thêm hoặc cập nhật cache khi database được cập nhật

**Ưu điểm:**
- Dữ liệu trong cache không bị cũ
- Đọc nhanh

**Nhược điểm:**
- Dữ liệu chưa có trong cache cho đến khi được thêm/cập nhật trong database
- Cache churn: nhiều dữ liệu được ghi vào cache nhưng không bao giờ được đọc
- Tăng chi phí ghi vì mỗi lần ghi cần hai lời gọi

### Cache Evictions và Time-to-Live (TTL) (Trang 155)

**Cache eviction xảy ra theo ba cách:**
1. Chủ động xóa item khỏi cache
2. Xóa item vì bộ nhớ đầy và item không được dùng gần đây (**LRU**)
3. Đặt **TTL** cho item

**TTL hữu ích cho:**
- Bảng xếp hạng
- Bình luận
- Dòng hoạt động

- TTL có thể từ vài giây đến vài giờ hoặc vài ngày
- Nếu bộ nhớ đầy khiến có quá nhiều eviction, hãy mở rộng theo chiều dọc hoặc chiều ngang

### Kết luận về Caching (Trang 156)

- **Lazy Loading** dễ triển khai, phù hợp với nhiều tình huống, đặc biệt ở phía đọc
- Thường kết hợp **Write-Through** với Lazy Loading cho các truy vấn/workload hưởng lợi từ cách tối ưu này
- Đặt TTL thường là lựa chọn tốt, trừ khi dùng Write-Through; hãy chọn giá trị phù hợp với ứng dụng
- Chỉ cache dữ liệu phù hợp, chẳng hạn hồ sơ người dùng hoặc blog
## Phần 2: Amazon Route 53 (Trang 157-187)

> *Chủ đề: DNS, DNS Records, Hosted Zones, Routing Policies và Health Checks.*

### DNS là gì? (Trang 158)

- **DNS** là viết tắt của Domain Name System, dịch hostname dễ đọc với con người thành địa chỉ IP của máy
- Ví dụ: `www.google.com` → `172.217.18.36`
- DNS là nền tảng của Internet
- DNS sử dụng cấu trúc đặt tên theo cấp bậc

> *Sơ đồ: .com → example.com → www.example.com → api.example.com*

### Thuật ngữ DNS (Trang 159)

| Thuật ngữ | Định nghĩa |
|---|---|
| **Domain Registrar** | Nhà đăng ký domain, ví dụ Amazon Route 53 hoặc GoDaddy |
| **DNS Records** | Các bản ghi DNS như A, AAAA, CNAME, NS |
| **Zone File** | Tệp chứa DNS records |
| **Name Server** | Máy chủ phân giải DNS query (authoritative hoặc non-authoritative) |
| **Top Level Domain (TLD)** | Miền cấp cao nhất, ví dụ .com, .us, .in, .gov, .org |
| **Second Level Domain (SLD)** | Miền cấp hai, ví dụ amazon.com hoặc google.com |

### DNS hoạt động như thế nào? (Trang 160)

> *Sơ đồ Trang 160: Web Browser hỏi Local DNS Server; Local DNS Server truy vấn Root DNS Server để tìm nameserver của `.com`, rồi TLD DNS Server để tìm nameserver của `example.com`, sau đó truy vấn SLD DNS Server (authoritative, do registrar quản lý) để lấy IP `9.10.11.12`. Local DNS Server cache kết quả theo TTL và trả IP cho browser; browser cũng có thể cache câu trả lời.*

### Amazon Route 53 (Trang 161)

- Dịch vụ DNS authoritative được quản lý toàn phần, có tính sẵn sàng và khả năng mở rộng cao
- **Authoritative** nghĩa là khách hàng có thể cập nhật DNS records
- Route 53 cũng là **Domain Registrar** và có thể kiểm tra tình trạng tài nguyên
- Theo slide nguồn, đây là dịch vụ AWS duy nhất có SLA availability 100%

### Route 53 - Records (Trang 162)

- Mỗi record gồm tên domain/subdomain, loại record, giá trị, **Routing Policy** và **TTL** (thời gian DNS Resolver lưu record trong cache)
- Các loại cần biết: **A, AAAA, CNAME, NS**; các loại nâng cao: **CAA, DS, MX, NAPTR, PTR, SOA, TXT, SPF, SRV**

### Route 53 - Các loại Record (Trang 163)

- **A** ánh xạ hostname sang IPv4; **AAAA** ánh xạ hostname sang IPv6
- **CNAME** ánh xạ hostname sang hostname khác; hostname đích phải có A hoặc AAAA record
- Không thể tạo CNAME ở đỉnh của DNS namespace (**Zone Apex**), ví dụ `example.com`; có thể tạo cho `www.example.com`
- **NS** liệt kê Name Servers của Hosted Zone, xác định tổ chức quản lý domain và cách định tuyến traffic cho domain

### Route 53 - Hosted Zones (Trang 164-165)

- **Hosted Zone** là vùng chứa các records mô tả cách định tuyến traffic cho một domain và các subdomain
- **Public Hosted Zone** chứa records định tuyến tên miền công khai trên Internet
- **Private Hosted Zone** chứa records định tuyến tên miền riêng tư bên trong một hoặc nhiều VPC
- Giá trên slide nguồn: **$0.50 mỗi tháng cho mỗi Hosted Zone**

> *Sơ đồ: Public Hosted Zone phân giải domain công khai; Private Hosted Zone phân giải domain nội bộ trong VPC.*

### Route 53 - TTL của Record (Trang 166)

- **TTL cao** (ví dụ 24 giờ): giảm số truy vấn Route 53 nhưng record cũ có thể còn trong cache lâu hơn
- **TTL thấp** (ví dụ 60 giây): tăng số truy vấn Route 53 nhưng thay đổi được nhận biết nhanh hơn
- Bắt buộc đặt TTL cho DNS records, ngoại trừ **Alias records**

### CNAME và Alias (Trang 167-168)

- **CNAME** trỏ hostname sang hostname khác, chỉ dùng cho domain không phải root (ví dụ `app.example.com`)
- **Alias** trỏ hostname đến AWS resource; dùng được cho cả root domain và subdomain, miễn phí theo slide nguồn và hỗ trợ health check native
- Alias tự nhận biết khi IP của resource thay đổi; với AWS resource, Alias luôn là record loại **A/AAAA** và không đặt TTL thủ công

### Route 53 - Routing Policies (Trang 169)

- Routing Policy xác định cách Route 53 trả lời DNS query
- DNS không định tuyến traffic như Load Balancer; DNS chỉ trả lời truy vấn phân giải tên
- Các policy: **Simple, Weighted, Failover, Latency-based, Geolocation, Multi-Value Answer, Geoproximity**

### Routing Policy - Simple (Trang 170)

- Thường định tuyến đến một resource; có thể chỉ định nhiều giá trị trong cùng record, khi đó client chọn ngẫu nhiên một giá trị
- Khi dùng Alias, chỉ định một AWS resource
- Không hỗ trợ Health Checks

### Routing Policy - Weighted (Trang 171)

- Phân chia tỷ lệ truy vấn đến từng resource; các records phải cùng tên và cùng loại
- Có thể gắn Health Checks; dùng để phân phối traffic giữa các Region hoặc thử phiên bản ứng dụng mới
- Đặt weight bằng 0 để ngừng gửi traffic tới resource đó; nếu tất cả weight đều bằng 0, mọi record được trả về với tỷ lệ ngang nhau

### Routing Policy - Latency-based (Trang 172)

- Trả lời bằng resource có latency thấp nhất theo phép đo traffic giữa người dùng và các AWS Region
- Phù hợp khi ưu tiên độ trễ; có thể kết hợp Health Checks để hỗ trợ failover

### Route 53 - Health Checks (Trang 173-174)

- HTTP Health Checks chỉ kiểm tra resource công khai
- Có thể theo dõi endpoint, **Calculated Health Checks** hoặc CloudWatch Alarm
- Health checkers nằm ngoài VPC nên không truy cập được endpoint riêng tư; với tài nguyên riêng tư, có thể tạo CloudWatch metric/alarm rồi gắn Health Check vào alarm

### Routing Policy - Failover (Active-Passive) (Trang 175)

> *Sơ đồ: Route 53 trả traffic về resource chính; khi Health Check báo resource chính không khỏe, DNS failover chuyển sang resource dự phòng.*

### Routing Policy - Geolocation (Trang 176)

- Định tuyến dựa trên vị trí người dùng, khác với Latency-based
- Có thể chọn theo quốc gia hoặc tiểu bang của Hoa Kỳ; nên tạo record **Default** khi không khớp vị trí
- Dùng cho nội dung theo khu vực, hạn chế phân phối nội dung hoặc cân bằng tải; có thể gắn Health Checks

### Routing Policy - Geoproximity (Trang 177-179)

- Định tuyến dựa trên vị trí địa lý của người dùng và resource; có thể điều chỉnh lượng traffic bằng **bias**
- Bias từ **1 đến 99** mở rộng vùng phục vụ; bias từ **-1 đến -99** thu hẹp vùng phục vụ
- Resource AWS được xác định bằng Region; resource ngoài AWS được xác định bằng vĩ độ/kinh độ
- Cần dùng **Route 53 Traffic Flow** cho policy này

> *Trang 178: bias 0 tại us-west-1 và us-east-1; Trang 179: tăng bias us-east-1 lên 50 để mở rộng vùng phục vụ về phía tây và thu hút thêm traffic.*

### Route 53 - Traffic Flow (Trang 180)

- Đơn giản hóa việc tạo và duy trì records trong cấu hình phức tạp bằng trình chỉnh sửa trực quan
- Có thể lưu cấu hình thành **Traffic Flow Policy**, áp dụng cho nhiều Hosted Zones và quản lý phiên bản

### Routing Policy - IP-based (Trang 181)

- Định tuyến dựa trên địa chỉ IP của client
- Cung cấp danh sách CIDR của client và endpoint/vị trí tương ứng (**user-IP-to-endpoint mappings**)
- Có thể tối ưu hiệu năng hoặc chi phí mạng, ví dụ định tuyến người dùng của một ISP đến endpoint cụ thể

### Routing Policy - Multi-Value (Trang 182)

- Dùng để trả về nhiều resource; mỗi query có thể trả về tối đa **8 records khỏe mạnh**
- Tương tự Simple nhưng hỗ trợ nhiều resource và Health Checks

### Domain Registrar và DNS Service (Trang 183-186)

- **Domain Registrar** là nơi đăng ký/mua domain, thường thu phí hằng năm; registrar thường cung cấp DNS service nhưng có thể dùng DNS provider khác
- Ví dụ: mua domain ở GoDaddy nhưng dùng Route 53 để quản lý DNS records
- Khi dùng registrar bên thứ ba với Route 53: tạo Hosted Zone trong Route 53, sau đó cập nhật **NS records** tại registrar để trỏ đến Name Servers của Route 53
- **Domain Registrar** không đồng nghĩa với **DNS Service**

> *Sơ đồ: Trước khi chuyển đổi, GoDaddy quản lý cả domain lẫn DNS; sau khi chuyển, GoDaddy vẫn quản lý domain còn Route 53 quản lý DNS.*

### Bài thực hành: Tạo RDS for MySQL (Trang 187)

- Tạo một database **Amazon RDS for MySQL**
