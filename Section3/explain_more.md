# Section 3 - Kiến thức mở rộng: RDS, Aurora, ElastiCache, Route 53


## 1. RDS (Relational Database Service)

### Multi-AZ và Read Replicas

#### Giải thích cốt lõi
**Multi-AZ** duy trì standby đồng bộ ở AZ khác và tự động failover qua cùng DNS endpoint để tăng tính sẵn sàng; nó không phân phối thao tác đọc. **Read Replicas** sao chép bất đồng bộ, có endpoint riêng để ứng dụng phân tải truy vấn đọc, có thể đặt khác AZ/Region và phải được promote khi cần dùng như database độc lập.

| Cấu hình | Replication | Mục đích chính |
|---|---|---|
| Multi-AZ | SYNC | High Availability và failover |
| Read Replica | ASYNC | Mở rộng đọc; có thể có replication lag |

#### NÊN làm (DO)
- Bật **Multi-AZ** cho database production cần HA, vì standby đồng bộ và failover tự động tốt hơn tự quản lý bản sao.
- Dùng **Read Replicas** cho workload đọc nhiều, báo cáo hoặc phân tích vì chúng tách tải đọc khỏi primary; cùng Region không tính phí truyền dữ liệu cross-AZ theo tài liệu nguồn.
- Cấu hình một Read Replica thành Multi-AZ khi cần dự phòng cho chính replica để DR → replica được triển khai với standby ở AZ khác; đây là bổ sung cho Read Replica chứ không biến nó thành cơ chế đồng bộ của primary.

#### KHÔNG NÊN làm (DON'T)
- Không dùng Multi-AZ để mở rộng khả năng đọc/ghi; nó không phục vụ phân tải đọc, nên dùng Read Replicas cho workload đọc.
- Không ghi vào Read Replica hoặc bỏ qua replication lag; truy vấn có thể thấy dữ liệu cũ, nên ghi vào primary và thiết kế ứng dụng chấp nhận độ trễ đọc khi phù hợp.

### Storage của RDS

#### Giải thích cốt lõi
**Storage Auto Scaling** tự tăng dung lượng storage khi database sắp hết chỗ, trong giới hạn **Maximum Storage Threshold**; nó xử lý thiếu dung lượng chứ không tự tăng IOPS. gp3 có giá thấp hơn gp2 20% với mức cơ bản 3.000 IOPS, gp2 phù hợp workload burst, còn io1/io2 dành cho nhu cầu Provisioned IOPS cao.

#### NÊN làm (DO)
- Bật **Storage Auto Scaling** khi dung lượng khó dự đoán, đặt Maximum Storage Threshold đủ cao và giám sát **FreeStorageSpace**; cách này an toàn hơn chờ đầy storage rồi sửa thủ công.
- Chọn gp3 cho workload phổ thông; chỉ chọn io1/io2 khi cần IOPS được cấp phát.

#### KHÔNG NÊN làm (DON'T)
- Không đặt Maximum Storage Threshold quá thấp hoặc coi Auto Scaling là cách tăng hiệu năng I/O; workload vẫn có thể lỗi khi hết hạn mức, nên dự báo dung lượng và chọn storage/IOPS phù hợp.

### Bảo mật RDS

#### Giải thích cốt lõi
Mã hóa **at rest** dùng KMS bảo vệ dữ liệu lưu trữ; **SSL/TLS** bảo vệ kết nối khi truyền; IAM Authentication, Security Groups và Secrets Manager lần lượt kiểm soát danh tính, mạng và credentials. Đây là các lớp bổ sung cho nhau, không thay thế lẫn nhau.

#### NÊN làm (DO)
- Bật **Encryption at Rest**, bắt buộc **SSL/TLS**, giới hạn truy cập bằng **VPC Security Groups**, dùng IAM Authentication khi phù hợp và lưu mật khẩu trong **Secrets Manager**; so với public access/default credentials, các biện pháp này giảm lộ dữ liệu và credentials.

#### KHÔNG NÊN làm (DON'T)
- Không để RDS public accessible, dùng credentials mặc định hoặc bỏ qua Parameter Groups; các cấu hình này có thể làm lộ database hoặc áp dụng sai thiết lập bảo mật, nên chỉ cấp quyền mạng/danh tính cần thiết và quản lý cấu hình tập trung.

### RDS Proxy

#### Giải thích cốt lõi
RDS Proxy gom và tái sử dụng các kết nối database để giảm số kết nối mở, CPU/RAM và timeout; dịch vụ được quản lý, tự mở rộng, đa AZ và có thể giảm thời gian failover. So với kết nối trực tiếp, proxy đặc biệt hữu ích khi ứng dụng tạo nhiều kết nối ngắn hạn.

#### NÊN làm (DO)
- Dùng RDS Proxy cho workload nhiều kết nối ngắn hạn, ứng dụng serverless/Lambda hoặc khi cần giảm thời gian failover; bật IAM Authentication để tránh quản lý credentials trong ứng dụng.

#### KHÔNG NÊN làm (DON'T)
- Không dùng khi cần giữ truy vấn chạy dài hơn 60 giây hoặc transaction kéo dài qua nhiều kết nối; proxy có thể không phù hợp với các phiên làm việc đó, nên dùng kết nối database trực tiếp.

---

## 2. Amazon Aurora

### Aurora và RDS

#### Giải thích cốt lõi
Aurora là database quan hệ độc quyền của AWS, tương thích PostgreSQL/MySQL, có storage tự mở rộng theo bước 10 GB (10 GB-128 TB theo tài liệu nguồn) và failover nhanh. Slide nêu mức hiệu năng gấp 5 lần MySQL và 3 lần PostgreSQL trên RDS, cùng chi phí cao hơn 20%; RDS thường đơn giản hơn và có thể phù hợp hơn khi đã có ứng dụng hiện hữu hoặc ngân sách hạn chế.

#### NÊN làm (DO)
- Chọn Aurora khi cần hiệu năng, storage tự mở rộng, failover nhanh, replication cross-Region hoặc Backtrack; chọn RDS khi cần cách quản lý đơn giản hơn hay muốn giữ ứng dụng MySQL/PostgreSQL hiện có.
- Dùng Aurora Serverless cho workload gián đoạn/khó dự đoán hoặc môi trường dev/test cần khả năng tự tạm dừng khi idle.

#### KHÔNG NÊN làm (DON'T)
- Không chọn Aurora chỉ vì cho rằng nó luôn rẻ hơn hoặc cần thiết cho mọi workload; chi phí theo slide cao hơn RDS 20%, nên so sánh hiệu năng/cost với RDS trước khi quyết định.

### Aurora Replicas

#### Giải thích cốt lõi
Aurora Replicas dùng chung storage với cluster, phục vụ truy vấn đọc và có thể được promote khi primary gặp sự cố. Slide nêu replica lag thường dưới 10 ms; đây là độ trễ thấp nhưng không đồng nghĩa mọi lần đọc đều tuyệt đối không trễ.

#### NÊN làm (DO)
- Tạo Aurora Replicas khi cần mở rộng đọc, hỗ trợ HA/failover hoặc phân bố workload giữa các Region; chúng phù hợp hơn việc gửi mọi truy vấn đọc đến primary.

#### KHÔNG NÊN làm (DON'T)
- Không giả định replica luôn nhất quán tức thời hoặc đọc từ replica cho dữ liệu bắt buộc mới nhất; hãy đọc từ writer endpoint khi cần dữ liệu mới nhất và thiết kế theo replication lag.

### Aurora Endpoints

#### Giải thích cốt lõi
Aurora endpoints là DNS names chọn DB instance cho từng kết nối; cluster/writer endpoint trỏ đến primary, reader endpoint cân bằng **kết nối** đọc giữa replicas (không cân bằng từng query), instance endpoint chỉ đến một instance cụ thể, còn custom endpoint đại diện nhóm instance do người vận hành chọn. Writer/reader endpoints cập nhật đích sau failover; client vẫn cần reconnect/retry.

| Endpoint | Dùng cho | Hành vi |
|---|---|---|
| **Cluster (writer) endpoint** | Đọc/ghi, đặc biệt mọi thao tác ghi | Trỏ đến primary hiện tại; cập nhật đích sau failover |
| **Reader endpoint** | Kết nối chỉ đọc | Cân bằng kết nối giữa Aurora Replicas; nếu chưa có replica thì kết nối đến primary |
| **Instance endpoint** | Chẩn đoán/tinh chỉnh một DB instance | Trỏ đến một instance cụ thể, không tự failover như cluster/reader endpoint |
| **Custom endpoint** | Một nhóm instance do người vận hành chọn | Cân bằng kết nối trong nhóm đã chọn |

#### NÊN làm (DO)
- Dùng cluster endpoint cho workload ghi và reader endpoint cho workload đọc; đây là lựa chọn phù hợp hơn việc tự quản lý địa chỉ instance.

#### KHÔNG NÊN làm (DON'T)
- Không dùng reader endpoint như thể nó phân phối từng query hoặc dùng instance endpoint làm đường kết nối chịu lỗi; mở kết nối mới khi cần phân tải đọc, và dùng cluster/reader endpoint khi cần chuyển hướng sau failover.

Nguồn chính thức: [Amazon Aurora endpoint connections](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.Endpoints.html)


## 3. Amazon ElastiCache

### Redis và Memcached

#### Giải thích cốt lõi
Redis và Memcached đều là cache trong bộ nhớ; Redis có replication, persistence và kiểu dữ liệu phong phú, còn Memcached tập trung vào cache key-value đơn giản, đa luồng và sharding. So sánh nhu cầu về HA, độ bền và cấu trúc dữ liệu trước khi chọn.

#### NÊN làm (DO)
- Chọn **Redis** khi cần tính sẵn sàng cao (replication, failover), lưu dữ liệu bền vững (AOF, RDB), kiểu dữ liệu nâng cao, Pub/Sub hoặc Lua scripting → các tính năng này phù hợp hơn Memcached cho cache có trạng thái và khả năng khôi phục.
- Chọn **Memcached** khi cần cache key-value đơn giản, đa luồng và sharding → tránh độ phức tạp của Redis khi không cần persistence hoặc replication.

**Gợi ý theo trường hợp sử dụng:**
| Trường hợp sử dụng | Nên chọn |
|----------|-------------|
| Lưu session | Redis |
| Cache kết quả truy vấn | Redis hoặc Memcached |
| Bảng xếp hạng thời gian thực | Redis (Sorted Sets) |
| Chat/nhắn tin | Redis (Pub/Sub) |
| Giới hạn tốc độ request | Redis |
| Cache HTML đơn giản | Memcached |
#### KHÔNG NÊN làm (DON'T)
- Không dùng Memcached khi cần persistence hoặc tự động failover → dữ liệu cache mất khi node lỗi; chọn Redis và cấu hình replication/persistence phù hợp.
- Không chọn Redis chỉ vì có nhiều tính năng nếu chỉ cần cache key-value đơn giản → tăng độ phức tạp và chi phí; cân nhắc Memcached.

### Chiến lược Caching

#### Giải thích cốt lõi
Với **Cache-Aside**, ứng dụng đọc cache trước, khi cache miss thì đọc database và nạp kết quả vào cache; **Write-Through** cập nhật cache cùng lúc với database. Cache-Aside ít ghi dữ liệu không được yêu cầu, còn Write-Through giảm nguy cơ đọc dữ liệu cũ sau thao tác ghi nhưng tăng chi phí ghi.

#### NÊN làm (DO)

**1. Cache-Aside (Lazy Loading):**
```python
def get_user(user_id):
    # 1. Kiểm tra cache trước
    user = cache.get(f"user:{user_id}")
    if user:
        return user
    
    # 2. Cache miss - lấy dữ liệu từ DB
    user = db.query("SELECT * FROM users WHERE id = ?", user_id)
    
    # 3. Lưu vào cache kèm TTL
    cache.set(f"user:{user_id}", user, ttl=3600)
    return user
```

**2. Write-Through:**
```python
def update_user(user_id, data):
    # 1. Ghi vào DB
    db.execute("UPDATE users SET ... WHERE id = ?", user_id, data)
    
    # 2. Cập nhật cache ngay
    user = db.query("SELECT * FROM users WHERE id = ?", user_id)
    cache.set(f"user:{user_id}", user, ttl=3600)
```

**Khi nào dùng từng chiến lược:**
| Chiến lược | Phù hợp nhất khi | Tránh dùng khi |
|----------|------------|------------|
| Cache-Aside | Đọc nhiều, chấp nhận dữ liệu cũ trong thời gian ngắn | Ghi nhiều, bắt buộc dữ liệu luôn mới |
| Write-Through | Cần dữ liệu mới, tần suất ghi có thể dự đoán | Ghi nhiều, nhiều key không bao giờ được đọc |

#### KHÔNG NÊN làm (DON'T)
- Không cache toàn bộ bảng → lãng phí bộ nhớ; chỉ cache dữ liệu được truy cập thường xuyên.
- Không đặt TTL quá dài cho dữ liệu thay đổi nhanh hoặc bỏ qua invalidation → người dùng có thể đọc dữ liệu cũ; dùng TTL ngắn hơn hoặc xóa/cập nhật cache khi dữ liệu nguồn đổi.
- Không dùng cache làm nguồn dữ liệu chính → eviction hoặc lỗi node có thể làm mất dữ liệu; lưu dữ liệu bền vững trong database.

### Thực hành tốt về TTL
#### Giải thích cốt lõi
TTL đặt thời gian một bản ghi cache được giữ trước khi hết hạn; TTL ngắn giảm thời gian dữ liệu cũ nhưng làm tăng cache miss, còn TTL dài cải thiện hit rate nhưng có thể trả dữ liệu stale. Quy tắc TTL bằng 0 khác nhau giữa các dịch vụ nên phải kiểm tra semantics của cache cụ thể.


#### NÊN làm (DO)
Đặt TTL theo yêu cầu về độ mới của dữ liệu:
- **Dữ liệu session**: 15-30 phút
- **Hồ sơ người dùng**: 1-24 giờ
- **Danh mục sản phẩm**: 1-24 giờ
- **Dữ liệu thời gian thực**: không cache hoặc dùng TTL rất ngắn

#### KHÔNG NÊN làm (DON'T)
- Không dùng TTL quá dài cho dữ liệu thường xuyên thay đổi → cache có thể trả dữ liệu cũ; giảm TTL hoặc chủ động invalidation.
- Không mặc định TTL bằng 0 hoặc vô hạn có cùng ý nghĩa ở mọi cache → item có thể không hết hạn, hết hạn ngay hoặc bị từ chối; xác nhận semantics của dịch vụ và đặt TTL rõ ràng.

### Loại bỏ item khỏi cache (Cache Eviction)
#### Giải thích cốt lõi
Khi bộ nhớ cache đầy, eviction policy quyết định item nào bị loại; Redis hỗ trợ các policy như LRU hoặc từ chối ghi (`noeviction`), trong khi Memcached tự loại item theo chính sách của nó. Eviction giải phóng bộ nhớ nhưng làm tăng cache miss; scale up thêm RAM, scale out thêm node và tăng phân tán dữ liệu.


#### NÊN làm (DO)
- Giám sát **CacheHitRate**, **CurrItems** và **evictions** → phát hiện sớm thiếu bộ nhớ hoặc tỷ lệ cache miss cao.
- Scale up khi thiếu RAM; scale out khi cần thêm dung lượng hoặc thông lượng, đồng thời chọn policy phù hợp với khả năng tái tạo dữ liệu.

#### KHÔNG NÊN làm (DON'T)
- Không chọn `noeviction` nếu ứng dụng không xử lý lỗi hết bộ nhớ → thao tác ghi mới có thể thất bại; dùng policy phù hợp hoặc tăng capacity.
- Không coi eviction là cơ chế lưu giữ dữ liệu → item bị loại phải có thể đọc lại từ nguồn bền vững; giữ dữ liệu chính trong database.

---

## 4. Route 53

### Các loại DNS Record
#### Giải thích cốt lõi
DNS record ánh xạ tên miền đến địa chỉ hoặc tên đích: A/AAAA trả địa chỉ IP, CNAME trỏ hostname khác, còn Route 53 Alias trỏ đến AWS resource và hỗ trợ zone apex. Chọn record type theo đích và việc có cần dùng tên miền gốc hay không.


#### NÊN làm (DO)
- Dùng A/AAAA cho địa chỉ IP, CNAME cho hostname không phải zone apex, và Alias cho AWS resource hoặc zone apex → tránh vi phạm giới hạn CNAME ở root domain; Alias tự theo dõi thay đổi IP của resource và không tính phí query theo tài liệu AWS.

#### KHÔNG NÊN làm (DON'T)
- Không tạo CNAME ở zone apex → DNS không cho phép CNAME coexist với các record bắt buộc khác tại root; dùng Alias nếu đích là AWS resource.
- Không dùng A/AAAA với IP cố định của AWS resource có thể đổi địa chỉ → record dễ lỗi khi IP thay đổi; dùng Alias.

### Routing Policies (Chính sách định tuyến)
#### Giải thích cốt lõi
Routing Policy quyết định Route 53 trả lời DNS query như thế nào; nó không phân phối trực tiếp từng request như Load Balancer. Simple trả record tĩnh; Weighted chia tỷ lệ câu trả lời; Latency chọn Region có độ trễ thấp; Failover dùng trạng thái health để chuyển primary/secondary; các policy địa lý và IP dựa vào vị trí hoặc dải IP của client.


#### NÊN làm (DO)
- Chọn policy theo mục tiêu: **Weighted** cho thử nghiệm/blue-green, **Latency-based** cho độ trễ thấp, **Geolocation** cho yêu cầu theo vị trí → mỗi policy chọn câu trả lời theo tiêu chí khác nhau; so với Simple, cân nhắc DNS caching/TTL và health-check behavior.

| Policy | Trường hợp sử dụng |
|--------|----------|
| **Simple** | Một resource |
| **Weighted** | A/B testing, triển khai blue-green |
| **Latency-based** | Chọn latency thấp nhất cho người dùng |
| **Failover** | HA active-passive |
| **Geolocation** | Nội dung theo khu vực, tuân thủ yêu cầu |
| **Geoproximity** | Điều phối traffic theo vị trí địa lý |
| **IP-based** | Định tuyến tùy chỉnh theo dải IP |
| **Multi-value** | Nhiều resource khỏe mạnh |

**Ví dụ Weighted Routing:**
```
- version-1.example.com: Weight 90 (90% traffic)
- version-2.example.com: Weight 10 (10% traffic)
```
#### KHÔNG NÊN làm (DON'T)
- Không dùng Weighted như thể nó đảm bảo phần trăm kết nối chính xác → DNS caching/TTL ảnh hưởng số client thực nhận mỗi endpoint; dùng nó để điều phối tương đối và theo dõi kết quả.
- Không nhầm Geolocation với Latency-based → một policy chọn theo vị trí, policy kia theo latency đo được; chọn theo yêu cầu thực tế.

### Health Checks (Kiểm tra tình trạng)
#### Giải thích cốt lõi
Route 53 Health Checks kiểm tra endpoint công khai, health check khác hoặc CloudWatch Alarm rồi cung cấp trạng thái để các policy như Failover loại record không khỏe khỏi câu trả lời. Health checker ở ngoài VPC nên không thể trực tiếp truy cập endpoint private; CloudWatch Alarm là lựa chọn thay thế cho tài nguyên riêng.


#### NÊN làm (DO)
- Bật Health Checks cho resource cần failover và chọn ngưỡng phù hợp → Route 53 có thể ngừng trả record không khỏe; dùng Calculated Health Checks hoặc CloudWatch Alarm khi cần tổng hợp trạng thái hay kiểm tra tài nguyên private.

#### KHÔNG NÊN làm (DON'T)
- Không trỏ health check trực tiếp đến địa chỉ private hoặc tắt kiểm tra cho resource cần failover → checker không thể đánh giá đúng hoặc Route 53 tiếp tục trả endpoint lỗi; dùng CloudWatch Alarm cho endpoint private và liên kết trạng thái với record.
- Không coi phản hồi redirect là lỗi mặc định → AWS xem mã 3xx là khỏe; kiểm tra cấu hình match code/string để tránh đánh dấu nhầm.

### Route 53 Resolver
#### Giải thích cốt lõi
Route 53 Resolver xử lý truy vấn DNS cho VPC; Resolver Endpoints mở luồng DNS hai chiều giữa VPC và mạng on-premises. Inbound nhận query từ on-premises, Outbound chuyển query từ VPC đến DNS server on-premises; đây là lựa chọn thay cho cấu hình DNS thủ công trên từng máy.


#### NÊN làm (DO)
- Dùng Inbound Endpoint khi on-premises cần phân giải tên trong VPC và Outbound Endpoint khi VPC cần phân giải tên on-premises → tách rõ hướng truy vấn, dễ quản lý bằng rules.

#### KHÔNG NÊN làm (DON'T)
- Không mở Resolver Endpoint rộng hơn các mạng cần thiết → có thể cho phép truy vấn DNS ngoài ý muốn; giới hạn Security Groups, rules và CIDR theo nguồn được phép.

---

## 5. Kiến trúc đọc nhiều với RDS và cache

### Luồng đọc và ghi qua ElastiCache và Aurora

#### Giải thích cốt lõi
Luồng đọc dùng cache trước rồi đến Reader Endpoint khi cache miss; luồng ghi đi qua Writer/Cluster Endpoint đến primary. Cache giảm truy vấn lặp lại nhưng có thể stale; Aurora endpoints chọn instance cho kết nối mới, không tự thiết lập lại kết nối đã mất sau failover.

#### NÊN làm (DO)
- Dùng cache cho dữ liệu đọc lặp lại và Reader Endpoint cho truy vấn đọc cần phân tải kết nối; gửi thao tác ghi qua Writer/Cluster Endpoint để giữ đúng vai trò primary.

#### KHÔNG NÊN làm (DON'T)
- Không giả định cache luôn đồng bộ tức thời hoặc endpoint tự retry connection → ứng dụng có thể trả dữ liệu cũ hoặc lỗi sau failover; đặt TTL/invalidation phù hợp và triển khai reconnect/retry an toàn.

---

