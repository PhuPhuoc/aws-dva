# Section 3 - DVA Practice Questions: RDS, Aurora, ElastiCache, Route 53

---

## 1. RDS & Aurora

### Question 1:
A company has a production MySQL database experiencing high read traffic. Which solution provides the BEST read scalability?

```
A. Enable Multi-AZ
B. Create Read Replicas
C. Increase the RDS instance size (vertical scaling)
D. Enable Amazon RDS Proxy
```

### ✅ Answer: **B. Create Read Replicas**

### 📖 Explanation (Vietnamese):

**Why Read Replicas:**
- **Read Replicas** cho phép **scale read capacity** lên đến **15 replicas**
- Replication **ASYNC** nên đọc từ replica có thể bị trễ; dữ liệu sẽ nhất quán cuối cùng (**eventually consistent**)
- Read queries có thể distribute across replicas
- Phù hợp cho **SELECT statements** (không phải writes)

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Multi-AZ | Chỉ cho **high availability**, không phải scaling |
| C. Vertical scaling | Tăng instance size, nhưng giới hạn về max size và downtime |
| D. RDS Proxy | Connection pooling, không tăng read capacity |

### 📝 Vocabulary Analysis:
- **Read Replica**: Database copy chỉ để đọc
- **ASYNC replication**: Replication không đồng bộ, có thể có lag
- **Horizontal scaling**: Scale out bằng cách thêm instances

### 🔄 Context Variations:
- **If need both HA and read scaling** → Read Replicas với Multi-AZ
- **If need write scaling** → Cân nhắc sharding/partitioning ở ứng dụng; Aurora cluster thông thường có một primary writer, không mặc định hỗ trợ nhiều writer

---

### Question 2:
What is the main difference between **RDS Multi-AZ** and **RDS Read Replicas**?

```
A. Multi-AZ uses ASYNC replication, Read Replicas use SYNC replication
B. Multi-AZ uses SYNC replication, Read Replicas use ASYNC replication
C. Both use the same replication type
D. Read Replicas cannot be in different AZs
```

### ✅ Answer: **B. Multi-AZ uses SYNC replication, Read Replicas use ASYNC replication**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Multi-AZ**: 
  - Uses **SYNC replication** 
  - Tự động failover khi primary fail
  - Một DNS name cho cả primary và standby
  - Primary và standby **luôn in sync**
  
- **Read Replicas**:
  - Uses **ASYNC replication**
  - Có thể promoted thành standalone DB
  - Replication lag có thể xảy ra
  - Dùng cho **read scaling**, không phải HA

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A | Ngược lại - Multi-AZ là SYNC, Read Replicas là ASYNC |
| C | Không đúng - chúng dùng different replication types |
| D | Read Replicas có thể within AZ, cross AZ, hoặc cross region |

---

### Question 3:
Which Aurora feature allows you to restore data to a specific point in time **without** using backups?

```
A. Backtrack
B. Point-in-time restore
C. Snapshot export
D. Replica lag
```

### ✅ Answer: **A. Backtrack**

### 📖 Explanation (Vietnamese):

**Why Backtrack:**
- **Backtrack** cho phép Aurora quay lại một thời điểm nằm trong khoảng thời gian lưu trữ đã cấu hình, không cần khôi phục từ snapshot backup. Tính năng này hữu ích để hoàn tác thay đổi ngoài ý muốn.

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| B. Point-in-time restore | Khôi phục về một thời điểm cụ thể bằng các bản sao lưu liên tục và transaction logs; đây không phải chức năng Backtrack và tạo một DB instance để khôi phục. |
| C. Snapshot export | Xuất dữ liệu ra S3, không phải chức năng khôi phục database. |
| D. Replica lag | Đo độ trễ replication, không phải chức năng khôi phục. |
 [Section3/dva_example.md#55F3]

### 📝 Vocabulary Analysis:
- **Backtrack**: Tính năng của Aurora để revert về thời điểm trước
- **Point-in-time restore**: Restore database về một thời điểm cụ thể
- **Storage volume**: Ổ lưu trữ dữ liệu của Aurora

---

### Question 4:
How many copies of data does Aurora maintain across how many Availability Zones?

```
A. 3 copies across 2 AZs
B. 4 copies across 3 AZs
C. 6 copies across 3 AZs
D. 6 copies across 6 AZs
```

### ✅ Answer: **C. 6 copies across 3 AZs**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- Aurora duy trì **6 copies** của data across **3 AZs**
- 4/6 copies cần thiết cho **writes**
- 3/6 copies cần thiết cho **reads**
- Self-healing với **peer-to-peer replication**
- Storage **striped across 100s of volumes**

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A | 3 copies là của standard RDS Multi-AZ, không phải Aurora |
| B | Không đúng số lượng copies |
| D | Aurora duy trì sáu bản sao dữ liệu trên ba AZ; không phải sáu AZ. |

---

## 2. ElastiCache

### Question 5:
Which caching strategy **only caches data that is requested** and lets the application recover from a **cache node failure** by loading the data again from the database?

```
A. Write-Through
B. Write-Behind
C. Lazy Loading (Cache-Aside)
D. Cache-Aside with Refresh-Ahead
```

### ✅ Answer: **C. Lazy Loading (Cache-Aside)**

### 📖 Explanation (Vietnamese):

**Why Lazy Loading:**
- **Lazy Loading** (Cache-Aside):
- Chỉ đưa dữ liệu vào cache khi có yêu cầu.
- Chỉ dữ liệu đã được yêu cầu mới được cache, không nạp sẵn dữ liệu chưa dùng.
- Lỗi cache node không làm ứng dụng ngừng hoạt động; chỉ làm tăng độ trễ trong lúc nạp lại cache.
- Khi cache node lỗi, cache miss sẽ đọc từ database rồi nạp lại dữ liệu vào cache.

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Write-Through | Write penalty, cache churn - data được write vào cache ngay cả khi không cần |
| B. Write-Behind | Async write, nhưng có thể mất data |
| D. Refresh-Ahead | Proactively refresh cached items, không phải lazy loading |

### 📝 Vocabulary Analysis:
- **Lazy Loading**: Chỉ load data vào cache khi cần
- **Cache miss**: Khi requested data không có trong cache
- **Cache hit**: Khi requested data có trong cache

### 🔄 Context Variations:
- **If data must never be stale** → Write-Through
- **If node failures should be transparent** → Lazy Loading

---

### Question 6:
A company needs to implement **user session storage** with automatic failover. Which ElastiCache option is most appropriate?

```
A. Memcached
B. Redis replication group with Multi-AZ and automatic failover
C. Redis (standalone)
D. ElastiCache Auto Discovery
```

### ✅ Answer: **B. Redis replication group with Multi-AZ and automatic failover**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- Cấu hình một Redis replication group có replica và bật **Multi-AZ automatic failover** để chuyển vai trò primary sang replica khi primary gặp sự cố
- Cluster mode là tùy chọn phân vùng dữ liệu (sharding), không phải điều kiện để bật automatic failover

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Memcached | Không cung cấp replication/failover tương đương Redis replication group |
| C. Redis standalone | Không có replica để promote khi node lỗi |
| D. Auto Discovery | Chỉ hỗ trợ tìm node/endpoint, không tự cung cấp HA |

### 📝 Vocabulary Analysis:
- **Replication group**: Nhóm Redis primary và replicas
- **Automatic failover**: Tự động promote replica khi primary gặp sự cố
- **Session storage**: Lưu trạng thái phiên người dùng


---

### Question 7:
What is the main difference between **Redis** and **Memcached** in ElastiCache?

```
A. Redis supports data persistence, Memcached does not
B. Redis is multi-threaded, Memcached is single-threaded
C. Redis cannot scale horizontally, Memcached can
D. Both have the same feature set
```

### ✅ Answer: **A. Redis supports data persistence, Memcached does not**

### 📖 Explanation (Vietnamese):

**Why A is correct:**
- **Redis**:
  - Supports **persistence** với **AOF** (Append Only File) và **RDB** snapshots
  - Có **Read Replicas** cho HA và read scaling
  - Supports **advanced data types** (Sets, Sorted Sets, Lists)
  
- **Memcached**:
  - **Non-persistent** - data mất khi restart
  - **No replication** - no high availability
  - **Multi-threaded** architecture
  - **Sharding** for horizontal scaling

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| B | Ngược lại - Redis single-threaded, Memcached multi-threaded |
| C | Redis có Cluster mode cho sharding, Memcached cũng có sharding |
| D | Feature sets khác nhau đáng kể |

---

## 3. Route 53

### Question 8:
Which Route 53 **routing policy** would you use to route traffic to the resource that provides the **lowest latency**?

```
A. Simple Routing Policy
B. Weighted Routing Policy
C. Latency Routing Policy
D. Geolocation Routing Policy
```

### ✅ Answer: **C. Latency Routing Policy**

### 📖 Explanation (Vietnamese):

**Why Latency Routing:**
- **Latency Routing Policy** trả lời bằng resource có latency thấp nhất theo phép đo giữa người dùng và các AWS Regions
- Phù hợp cho ứng dụng triển khai ở nhiều Region; resource gần nhất về địa lý không nhất thiết luôn có latency thấp nhất

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Simple | Route đến single resource, không có intelligence |
| B. Weighted | Phân phối traffic theo percentage, không dựa trên latency |
| D. Geolocation | Route dựa trên user location, không phải latency |

### 📝 Vocabulary Analysis:
- **Latency**: Độ trễ mạng
- **Routing Policy**: Rules quyết định cách Route 53 respond DNS queries
- **Region**: AWS geographic area

### 🔄 Context Variations:
- **If need A/B testing** → Weighted Policy
- **If need regional restrictions** → Geolocation Policy
- **If need failover** → Failover Policy

---

### Question 9:
Which DNS record type is used to map a **root domain** (e.g., example.com) to an **AWS resource** (e.g., CloudFront distribution)?

```
A. CNAME record
B. A record
C. Alias record
D. MX record
```

### ✅ Answer: **C. Alias record**

### 📖 Explanation (Vietnamese):

**Why Alias record:**
- **Alias record** là AWS-specific extension cho Route 53
- Cho phép map **root domain** (apex domain) đến AWS resources
- CNAME **không thể** dùng cho root domain (RFC compliant)
- Alias record **không charge** cho queries
- Tự động resolve IP changes của AWS resources

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. CNAME | Không cho phép root domain (chỉ subdomains) |
| B. A record | Map đến IPv4 address, không phải AWS resource |
| D. MX record | Dùng cho email routing, không phải general routing |

### 📝 Vocabulary Analysis:
- **Root domain/Apex domain**: Domain không có subdomain (example.com)
- **Alias record**: Route 53 specific record type
- **CNAME**: Canonical name record

---

### Question 10:
What capability can Route 53 health checks provide when they are associated with DNS routing records?

```
A. To monitor the health of Route 53 DNS service itself
B. Enable DNS failover by evaluating resource health and returning healthy records
C. To send alerts when resources become unavailable
D. To backup Route 53 zone files
```

### ✅ Answer: **B. Enable DNS failover by evaluating resource health and returning healthy records**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- Health Check đánh giá tình trạng endpoint/alarm; khi được liên kết với DNS record có health evaluation, nó cho phép DNS failover tránh trả về resource không khỏe
- HTTP Health Checks chỉ kiểm tra public resources; với endpoint private có thể dùng CloudWatch Alarm
- Có thể theo dõi endpoint, CloudWatch Alarm hoặc Calculated Health Checks

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A | Route 53 là managed service, không cần check chính nó |
| C | Health checks không gửi alerts (cần CloudWatch cho alerts) |
| D | Health checks không backup data |

### 📝 Vocabulary Analysis:
- **Health Check**: Kiểm tra trạng thái của một endpoint
- **DNS failover**: Tự động chuyển sang resource khác khi primary fail
- **Unhealthy threshold**: Số lần check thất bại trước khi marked unhealthy

---

## 4. Combined Architecture Questions

### Question 11:
A production database uses RDS. Which RDS feature provides automatic database failover to a standby instance?

A. RDS Read Replica
B. Amazon RDS Proxy
C. RDS Multi-AZ
D. RDS Storage Auto Scaling

### ✅ Answer: **C. RDS Multi-AZ**

### 📖 Explanation (Vietnamese):

**Why RDS Multi-AZ:**
- **RDS Multi-AZ** cung cấp **automatic failover** cho database
- Khi primary instance fail:
  - Standby được promoted thành primary
- DB endpoint được cập nhật để trỏ đến primary mới sau failover; ứng dụng cần reconnect/retry để thiết lập kết nối mới

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Read Replica | Dùng để mở rộng khả năng đọc, không tự động failover primary của database |
| B. RDS Proxy | Quản lý/pool kết nối, không thay thế Multi-AZ cho failover |
| D. Storage Auto Scaling | Tăng storage khi cần, không xử lý failover instance |

### 🔄 Context Variations:
- **Nếu cần mở rộng khả năng đọc và failover** → Aurora với Read Replicas
- **Nếu cần failover cross-Region** → Cross-Region Read Replica được sao chép bất đồng bộ; cần promote rồi chuyển ứng dụng/DNS sang endpoint mới, hoặc cấu hình riêng một cơ chế DR khác

---

### Question 12:
Which AWS services work together to provide a **serverless, auto-scaling, highly available** web application architecture?

```
A. EC2 + RDS + Route 53
B. Lambda + API Gateway + DynamoDB
C. ECS + ElastiCache + RDS
D. Lightsail + S3 + CloudFront
```

### ✅ Answer: **B. Lambda + API Gateway + DynamoDB**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Lambda**: Serverless compute, tự động scale, pay per invocation
- **API Gateway**: Serverless API management, throttles, caching
- **DynamoDB**: Serverless database, auto scaling, multi-AZ replication
- Tất cả đều **managed services**, **serverless**, **auto-scaling**

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. EC2 + RDS | Cần manage servers, không serverless |
| C. ECS + ElastiCache + RDS | Cần manage containers và servers |
| D. Lightsail + S3 + CloudFront | Lightsail là VMs, không phải serverless |

### 📝 Vocabulary Analysis:
- **Serverless**: Không cần manage servers, AWS tự động scale
- **Auto-scaling**: Tự động tăng/giảm capacity
- **Managed services**: AWS quản lý infrastructure

---

### Question 13:
A company is experiencing **high latency** when users read data from the database. The data changes infrequently. Which caching solution should be implemented?

```
A. Write-Through cache to cache every write
B. Lazy Loading (Cache-Aside) pattern with ElastiCache
C. Cache every query result indefinitely
D. Increase RDS instance size
```

### ✅ Answer: **B. Lazy Loading (Cache-Aside) pattern with ElastiCache**

### 📖 Explanation (Vietnamese):

**Why Lazy Loading is best:**
- **Lazy Loading** is ideal when:
  - Data changes **infrequently**
  - Same data được read nhiều lần
  - Chỉ requested data được cached (efficient)
- ElastiCache Redis/Memcached provide **low latency** (< 1ms)
- Cache có TTL để tránh stale data
- Đọc từ cache trước, DB khi miss

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Write-Through | Unnecessary overhead nếu data rarely read |
| C. Cache indefinitely | Risk of stale data, không recommend |
| D. Increase RDS size | Vertical scaling tốn kém, không phải optimal solution |

### 📝 Vocabulary Analysis:
- **Cache hit**: Data found in cache
- **Cache miss**: Data not in cache, need to fetch from DB
- **Cache-aside**: Application manages cache explicitly

### 🔄 Context Variations:
- **If data changes frequently** → Write-Through có thể better
- **If need instant consistency** → No cache, direct DB access

---

## 5. Route 53 DNS resolution (Source Page 160)

### Question 14:
A user enters `example.com`. Neither the browser nor its **local DNS server** has a cached answer. Which sequence resolves the hostname?

```
A. Browser → TLD DNS server → Root DNS server → authoritative DNS server
B. Browser → local DNS server → Root DNS server → TLD DNS server → authoritative DNS server
C. Browser → domain registrar → Root DNS server → TLD DNS server
D. Browser → authoritative DNS server directly, without a DNS resolver
```

### ✅ Answer: **B**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Browser gửi query đến **Local DNS Server**. Nếu không có cache, Local DNS Server hỏi **Root DNS Server** để tìm nameserver của `.com`.
- **TLD DNS Server** trả về nameserver có thẩm quyền cho `example.com`; **SLD/authoritative DNS Server** trả về địa chỉ IP của hostname.
- Local DNS Server cache câu trả lời theo **TTL**, rồi trả IP về browser. Đây là luồng được minh họa ở Trang 160.

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| A | Đảo thứ tự Root và TLD; resolver cần tìm máy chủ của TLD trước khi hỏi TLD. |
| C | Domain Registrar không nằm trong mỗi lần truy vấn phân giải hostname; Root DNS được hỏi trước TLD DNS. |
| D | Browser thường gửi query đến Local DNS Server thay vì trực tiếp truy vấn authoritative server. |

### 📝 Phân tích từ vựng:
- **Local DNS Server / Resolver**: máy chủ nhận và thay mặt client phân giải DNS query.
- **TLD DNS Server**: máy chủ cho miền cấp cao nhất như `.com`.
- **Authoritative DNS Server**: máy chủ giữ DNS records chính thức cho domain.

---

## Summary Table

| Topic | Key Points |
|-------|------------|
| **RDS Multi-AZ** | SYNC replication, automatic failover, HA |
| **RDS Read Replicas** | ASYNC replication, read scaling, 15 replicas |
| **Aurora** | 6 copies/3 AZ, Backtrack, auto-storage scaling |
| **ElastiCache** | Redis vs Memcached, Lazy Loading vs Write-Through |
| **Route 53** | Routing policies, Health Checks, Alias records |
