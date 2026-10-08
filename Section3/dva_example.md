# Section 3 - DVA Practice Questions: ELB, ASG, RDS, Aurora, ElastiCache, Route 53

---

## 1. Elastic Load Balancer

### Load Balancer Types and Selection

### Question 1:
An application needs to route traffic based on the URL path (e.g., `/api` goes to one set of servers, `/web` goes to another). Which Load Balancer type should be used?

```
A. Network Load Balancer (NLB)
B. Application Load Balancer (ALB)
C. Classic Load Balancer (CLB)
D. Gateway Load Balancer
```

### ✅ Answer: **B. Application Load Balancer (ALB)**

### 📖 Explanation (Vietnamese):

**Why choose ALB:**
- **ALB** là Layer 7 (HTTP/HTTPS) Load Balancer, hỗ trợ **path-based routing** (`/api`, `/web`)
- ALB có thể route dựa trên:
  - URL path
  - Hostname
  - Query string
  - HTTP headers
- NLB chỉ hoạt động ở Layer 4 (TCP/UDP), không hỗ trợ routing dựa trên URL

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. NLB | Layer 4, không hỗ trợ path-based routing |
| C. CLB | Deprecated, không hỗ trợ advanced routing features |
| D. Gateway LB | Dùng cho network virtual appliances, không phải HTTP routing |

### 📝 Vocabulary Analysis:
- **Path-based routing**: routing dựa trên đường dẫn URL
- **Layer 7**: Application layer trong OSI model
- **Target group**: Nhóm các resources nhận traffic

### 🔄 Context Variations:
- **If routing by hostname** → Still ALB (host-based routing)
- **If millions of requests, low latency** → NLB instead
- **If routing TCP/UDP traffic** → NLB or CLB

---

### Question 2:
A company needs to expose a static IP address that can be whitelisted by their partners. Which Load Balancer type meets this requirement?

```
A. Application Load Balancer
B. Network Load Balancer
C. Classic Load Balancer
D. Gateway Load Balancer
```

### ✅ Answer: **B. Network Load Balancer**

### 📖 Explanation (Vietnamese):

**Why choose NLB:**
- **NLB** cung cấp **một static IP per Availability Zone**
- Hỗ trợ **Elastic IP (EIP)** có thể attach đến NLB
- Partner có thể whitelist static IP thay vì hostname
- NLB hoạt động ở **Layer 4**, phù hợp cho TCP/UDP traffic

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. ALB | Cung cấp fixed hostname (XXX.region.elb.amazonaws.com), không có static IP |
| C. CLB | Deprecated, không hỗ trợ EIP |
| D. Gateway LB | Dùng cho transparent network gateway, không phải static IP |

### 📝 Vocabulary Analysis:
- **Elastic IP (EIP)**: Static IPv4 address có thể attach/detach
- **Whitelist**: Danh sách IP được phép truy cập
- **Layer 4**: Transport layer trong OSI model

---

### Question 3:
A web application uses HTTPS and needs to serve multiple domains with SSL certificates. Which feature allows the Load Balancer to select the correct certificate?

```
A. Connection Draining
B. Cross-Zone Load Balancing
C. Server Name Indication (SNI)
D. Sticky Sessions
```

### ✅ Answer: **C. Server Name Indication (SNI)**

### 📖 Explanation (Vietnamese):

**Why choose SNI:**
- **SNI** cho phép server host **multiple SSL certificates** trên cùng một IP
- Khi client gửi request, nó gửi kèm **hostname** trong TLS handshake
- Load Balancer đọc hostname và chọn đúng certificate
- Được hỗ trợ bởi **ALB và NLB** (newer generation)

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Connection Draining | Chờ hoàn thành requests khi instance deregister |
| B. Cross-Zone LB | Phân phối traffic đều across AZs |
| D. Sticky Sessions | Bind client to same instance |

### 📝 Vocabulary Analysis:
- **Server Name Indication (SNI)**: Extension của TLS cho phép client specify hostname
- **SSL Certificate**: Digital certificate xác thực identity
- **TLS handshake**: Quá trình thiết lập secure connection

### 🔄 Context Variations:
- **If using CLB** → SNI không hoạt động (CLB deprecated)
- **If only one domain** → Default certificate cũng đủ

---

### Question 4:
Which statement about **Cross-Zone Load Balancing** is correct?

```
A. It is enabled by default for Network Load Balancers
B. Application Load Balancer charges for inter-AZ data transfer when enabled
C. Network Load Balancer charges for inter-AZ data transfer when enabled
D. Cross-Zone LB cannot be disabled for Application Load Balancers
```

### ✅ Answer: **C. Network Load Balancer charges for inter-AZ data transfer when enabled**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **NLB**: Disabled by default, **có charge** cho inter-AZ data transfer khi enabled
- **ALB**: Enabled by default, **không charge** cho inter-AZ data

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A | NLB disabled by default, không phải enabled |
| B | ALB không charge cho inter-AZ data |
| D | ALB có thể disable ở Target Group level |

---

## 2. Auto Scaling Group

### Question 5:
A company wants to automatically scale their application based on the average CPU utilization, keeping it around 50%. Which scaling policy type is most appropriate?

```
A. Simple Scaling
B. Step Scaling
C. Target Tracking Scaling
D. Scheduled Scaling
```

### ✅ Answer: **C. Target Tracking Scaling**

### 📖 Explanation (Vietnamese):

**Why choose Target Tracking Scaling:**
- **Target Tracking Scaling** là **strongly recommended** cho việc scale dựa trên metric target
- Set target value (ví dụ: CPU 50%) và ASG tự động scale out/in
- AWS tự quản lý scaling actions dựa trên metric
- Đơn giản và hiệu quả cho hầu hết use cases

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Simple Scaling | Cần manual CloudWatch alarm, ít linh hoạt |
| B. Step Scaling | Cho multiple step adjustments, phức tạp hơn cần thiết |
| D. Scheduled Scaling | Cho predictable patterns, không phải reactive scaling |

### 📝 Vocabulary Analysis:
- **Target Tracking Scaling**: Tự động scale để đạt target metric value
- **CPU Utilization**: Percentage of CPU đang được sử dụng
- **Scaling policy**: Rules xác định khi nào scale out/in

### 🔄 Context Variations:
- **If scaling based on time** (e.g., 5PM daily) → Scheduled Scaling
- **If need fine-grained control** → Step Scaling

---

### Question 6:
An ASG has a cooldown period of 300 seconds. After a scale-out event, how long will the ASG wait before launching another instance?

```
A. 0 seconds - can launch immediately
B. 300 seconds
C. 600 seconds
D. Until CPU drops below threshold
```

### ✅ Answer: **B. 300 seconds**

### 📖 Explanation (Vietnamese):

**Why 300 seconds is correct:**
- **Cooldown period** là thời gian ASG **chờ trước khi thực hiện** scaling activity khác
- Default cooldown là **300 seconds**
- Trong thời gian này, ASG không launch hoặc terminate instances
- Cho phép metrics **stabilize** trước khi có thêm changes

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A | Cooldown ngăn launch immediately |
| C | 600 seconds = 2x default, không phải correct |
| D | Cooldown là fixed time, không phải metric-based |

### 📝 Vocabulary Analysis:
- **Cooldown period**: Thời gian chờ giữa các scaling activities
- **Scale-out**: Thêm instances để tăng capacity
- **Scale-in**: Xóa instances để giảm capacity

### 🔄 Context Variations:
- **If using pre-baked AMI** → Có thể reduce cooldown vì instances boot nhanh hơn
- **If scale-in too aggressive** → Increase cooldown

---

### Question 7:
What happens during an **Instance Refresh** in an Auto Scaling Group?

```
A. EC2 instances are paused and memory is swapped
B. All instances are terminated and new ones are launched from the Launch Template
C. The Launch Template is updated and instances are re-created progressively
D. Only the Auto Scaling Group configuration is updated
```

### ✅ Answer: **C. The Launch Template is updated and instances are re-created progressively**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **Instance Refresh** cho phép cập nhật Launch Template và **re-create instances** một cách từ từ
- Instances được replaced **progressively** (một số tại một thời điểm)
- Có thể set **minimum healthy percentage** (recommend 90%)
- Có thể set **warm-up time** để instance ready trước khi nhận traffic

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A | Không pause instances, đây không phải how it works |
| B | Instances được replaced progressively, không phải all at once |
| D | Instance Refresh thay đổi instances, không phải chỉ configuration |

---

## 3. RDS & Aurora

### Question 8:
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
- Replication là **ASYNC** nên reads cuối cùng sẽ consistent
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
- **If need write scaling** → Sharding or Aurora (multiple writers)

---

### Question 9:
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

### Question 10:
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
- **Backtrack** là feature độc đáo của **Aurora** cho phép:
  - Restore data **đến bất kỳ thời điểm nào** trong quá khứ
  - **Không cần backup** để sử dụng
  - Working directly với data trên storage
  - Rất hữu ích cho việc undo mistakes

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| B. Point-in-time restore | Cần backup (snapshot) để restore |
| C. Snapshot export | Export data ra S3, không phải restore feature |
| D. Replica lag | Đo độ trễ replication, không phải restore feature |

### 📝 Vocabulary Analysis:
- **Backtrack**: Tính năng của Aurora để revert về thời điểm trước
- **Point-in-time restore**: Restore database về một thời điểm cụ thể
- **Storage volume**: Ổ lưu trữ dữ liệu của Aurora

---

### Question 11:
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
| D | Không có 6 AZs, chỉ 3 AZs |

---

## 4. ElastiCache

### Question 12:
Which caching strategy **only caches requested data** and is **not affected by node failures**?

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
  - Chỉ cache data khi **có request**
  - Only requested data được cached, không fill cache với unused data
  - **Node failures không fatal** - chỉ tăng latency để warm cache
  - Nếu node fail, cache miss sẽ get từ DB và repopulate

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

### Question 13:
A company needs to implement **user session storage** with automatic failover. Which ElastiCache option is most appropriate?

```
A. Memcached
B. Redis Cluster Mode
C. Redis (standalone)
D. ElastiCache Auto Discovery
```

### ✅ Answer: **B. Redis Cluster Mode**

### 📖 Explanation (Vietnamese):

**Why Redis Cluster Mode:**
- **Redis Cluster Mode** cung cấp:
  - **High availability** với automatic failover
  - Data được replicated across multiple nodes
  - Nếu primary node fail, replica được promoted
  - **Session data được preserve** khi node fail
  - Hỗ trợ **Multi-AZ** deployment

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Memcached | Không có replication hoặc failover support |
| C. Redis standalone | Không có automatic failover - single point of failure |
| D. Auto Discovery | Connection feature, không phải HA solution |

### 📝 Vocabulary Analysis:
- **Cluster Mode**: Phân phối data across multiple nodes
- **Automatic failover**: Tự động chuyển sang replica khi primary fail
- **Session storage**: Lưu trữ user session state

---

### Question 14:
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

## 5. Route 53

### Question 15:
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
- **Latency Routing Policy** route traffic đến resource có **lowest latency** cho user
- Route 53 kiểm tra latency từ user đến các regions/resources
- Tự động chọn resource gần nhất hoặc có response nhanh nhất
- Phù hợp cho applications deployed ở nhiều regions

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

### Question 16:
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

### Question 17:
What is the purpose of a Route 53 **Health Check**?

```
A. To monitor the health of Route 53 DNS service itself
B. To automatically remove unhealthy resources from DNS responses
C. To send alerts when resources become unavailable
D. To backup Route 53 zone files
```

### ✅ Answer: **B. To automatically remove unhealthy resources from DNS responses**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Health Checks** kiểm tra health của resources định kỳ
- Nếu resource **unhealthy**, Route 53 **tự động loại bỏ** khỏi DNS responses
- Traffic được route đến healthy resources
- Có thể check:
  - Endpoint (IP hoặc domain)
  - CloudWatch alarm
  - Calculated health checks

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

## 6. Combined Architecture Questions

### Question 18:
A company needs a **highly available web application** with:
- Traffic distributed across multiple AZs
- Automatic scaling based on CPU
- Database with automatic failover
- Low latency for global users

Which architecture component provides **automatic failover** for the database?

```
A. Auto Scaling Group
B. Application Load Balancer
C. RDS Multi-AZ
D. Route 53 Health Check
```

### ✅ Answer: **C. RDS Multi-AZ**

### 📖 Explanation (Vietnamese):

**Why RDS Multi-AZ:**
- **RDS Multi-AZ** cung cấp **automatic failover** cho database
- Khi primary instance fail:
  - Standby được promoted thành primary
  - DNS endpoint tự động chuyển
  - **Application reconnect** tự động
- No manual intervention required
- SYNC replication đảm bảo zero data loss

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. ASG | Tự động scale instances, không phải database failover |
| B. ALB | Phân phối traffic, không có database failover |
| D. Route 53 Health Check | Monitor health, nhưng không tự động failover database |

### 🔄 Context Variations:
- **If need read scaling + failover** → Aurora với Read Replicas
- **If need cross-region failover** → RDS Read Replica as standby

---

### Question 19:
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

### Question 20:
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

## Summary Table

| Topic | Key Points |
|-------|------------|
| **ALB** | Layer 7, path/host routing, SNI, X-Forwarded-For |
| **NLB** | Layer 4, static IP, EIP, extreme performance |
| **ASG** | Launch Template, Target Tracking, Cooldown, Instance Refresh |
| **RDS Multi-AZ** | SYNC replication, automatic failover, HA |
| **RDS Read Replicas** | ASYNC replication, read scaling, 15 replicas |
| **Aurora** | 6 copies/3 AZ, Backtrack, auto-storage scaling |
| **ElastiCache** | Redis vs Memcached, Lazy Loading vs Write-Through |
| **Route 53** | Routing policies, Health Checks, Alias records |
