# Section 3 - Scalability & High Availability: ELB, ASG | RDS, Aurora & ElastiCache | Route 53

---

## Phần 1: Scalability & High Availability (Trang 88-93)

### Scalability & High Availability Overview (Trang 88-89)

- **Scalability** nghĩa là một application/system có thể xử lý tải lớn hơn bằng cách thích ứng
- Có hai loại scalability:
  - **Vertical Scalability** (Scale up/down)
  - **Horizontal Scalability** (= elasticity) (Scale out/in)
- Scalability thường đi đôi với **High Availability**

### Vertical Scalability (Trang 89)

- Vertical scalability (= **scale up/down**) nghĩa là tăng kích thước của instance
- Thường dùng cho các **non-distributed systems**, ví dụ như database
- RDS, ElastiCache là các services có thể scale vertically

> *Hình minh họa: EC2 Instance M5.large (2 vCPUs/8 GB) → T2.micro (1 vCPU/1 GB)*

### Horizontal Scalability (Trang 90)

- Horizontal Scalability (= **scale out/in**) nghĩa là tăng số lượng instances/systems cho application
- Horizontal scaling ám chỉ **distributed systems**
- Rất phổ biến cho **web applications** và **modern applications**
- Sử dụng **Auto Scaling Group**

> *Hình minh họa: Nhiều EC2 Instances T2.micro được thêm vào qua Auto Scaling Group*

### High Availability (Trang 91-92)

- High Availability thường đi đôi với horizontal scaling
- High availability nghĩa là chạy application/system ở ít nhất **2 data centers (Availability Zones)**
- Mục tiêu của high availability là **survive a data center loss**
- High availability có thể là:
  - **Passive** (ví dụ: RDS Multi AZ)
  - **Active** (ví dụ: horizontal scaling)

### High Availability & Scalability cho EC2 (Trang 92-93)

- **Vertical Scaling**: Tăng instance size (= scale up/down)
  - Từ: t2.nano (0.5 GB RAM, 1 vCPU) → u-12tb1.metal (12.3 TB RAM, 448 vCPUs)
- **Horizontal Scaling**: Tăng số lượng instances (= scale out/in)
  - Auto Scaling Group
  - Load Balancer
- **High Availability**: Chạy instances cho cùng một application qua nhiều AZ
  - Auto Scaling Group multi AZ
  - Load Balancer multi AZ

---

## Phần 2: Elastic Load Balancer - ELB (Trang 94-118)

### What is Load Balancing? (Trang 94)

- **Load Balancers** là servers forward traffic đến nhiều servers (ví dụ: EC2 instances) downstream

> *Sơ đồ: Client → Load Balancer → Multiple EC2 Instances*

### Why use a Load Balancer? (Trang 95-96)

- **Spread load** across multiple downstream instances
- **Expose a single point of access** (DNS) to your application
- **Seamlessly handle failures** of downstream instances
- **Do regular health checks** to your instances
- **Provide SSL termination** (HTTPS) for your websites
- **Enforce stickiness** with cookies
- **High availability** across zones
- **Separate public traffic** from private traffic

### Why use an Elastic Load Balancer? (Trang 96)

- **Managed load balancer** - AWS đảm bảo nó sẽ hoạt động
- AWS tự lo **upgrades, maintenance, high availability**
- AWS cung cấp **few configuration knobs**
- Tích hợp với nhiều AWS offerings/services:
  - EC2, EC2 Auto Scaling Groups, Amazon ECS
  - AWS Certificate Manager (ACM), CloudWatch
  - Route 53, AWS WAF, AWS Global Accelerator

### Health Checks (Trang 97-98)

- Health Checks **rất quan trọng** cho Load Balancers
- Cho phép load balancer biết instances có sẵn sàng reply requests không
- Health check được thực hiện trên **port** và **route** (`/health` là phổ biến)
- Nếu response **không phải 200 (OK)**, instance được coi là **unhealthy**

### Load Balancer Security Groups (Trang 98)

> *Sơ đồ: Security Groups configuration*
> - Load Balancer Security Group: Allow traffic from anywhere
> - Application Security Group: Allow traffic only from Load Balancer

### Types of Load Balancer on AWS (Trang 99)

| Feature | Application Load Balancer | Network Load Balancer | Gateway Load Balancer | Classic Load Balancer |
|---------|-------------------------|----------------------|----------------------|----------------------|
| **Layer** | Layer 7 | Layer 4 | Layer 3 Gateway + Layer 4 LB | Layer 4/7 |
| **Target type** | IP, Instance, Lambda | IP, Instance, ALB | IP, Instance | IP, Instance |
| **Protocol listeners** | HTTP, HTTPS, gRPC | TCP, UDP, TLS | IP | TCP, SSL/TLS, HTTP, HTTPS |

### Application Load Balancer (ALB) - Layer 7 (Trang 100-102)

- ALB là **Layer 7 (HTTP)** của OSI model (hỗ trợ HTTP/HTTPS)
- Load balancing đến **nhiều HTTP applications** across machines (target groups)
- Load balancing đến **nhiều applications trên cùng một machine** (ví dụ: containers)
- Hỗ trợ **HTTP/2** và **WebSocket**
- Hỗ trợ **redirects** (ví dụ: từ HTTP sang HTTPS)

**Routing tables đến các target groups khác nhau:**
- Routing dựa trên **path** trong URL (`example.com/users` & `example.com/posts`)
- Routing dựa trên **hostname** trong URL (`one.example.com` & `other.example.com`)
- Routing dựa trên **Query String, Headers** (`example.com/users?id=123&order=false`)
- ALB phù hợp cho **micro services** & **container-based applications**
- Có **port mapping feature** để redirect đến dynamic port trong ECS

### ALB Target Groups (Trang 102-103)

- **EC2 instances** (có thể được quản lý bởi Auto Scaling Group) - HTTP
- **ECS tasks** (được quản lý bởi ECS) - HTTP
- **Lambda functions** - HTTP request được translate thành JSON event
- **IP Addresses** - phải là private IPs

**Lưu ý:**
- ALB có thể route đến **nhiều target groups**
- Health checks được thực hiện ở **target group level**

### ALB - Good to Know (Trang 105)

- **Fixed hostname** (XXX.region.elb.amazonaws.com)
- Application servers **không thấy trực tiếp IP** của client
- **True IP** của client được chèn vào header **X-Forwarded-For**

### Network Load Balancer (NLB) - Layer 4 (Trang 106-107)

- **Layer 4** cho phép forward **TCP & UDP traffic** đến instances
- Xử lý **millions of requests per seconds**
- **Less latency** ~100 ms (so với 400 ms của ALB)
- NLB có **một static IP per AZ**, hỗ trợ **Elastic IP** (hữu ích cho whitelisting)
- Dùng cho **extreme performance, TCP or UDP traffic**
- **Không nằm trong AWS free tier**

**Target Groups cho NLB:**
- EC2 instances
- IP Addresses (phải là private IPs)
- Application Load Balancer
- Health Checks hỗ trợ **TCP, HTTP và HTTPS Protocols**

### Gateway Load Balancer (Trang 109-110)

- Deploy, scale, và manage fleet của **3rd party network virtual appliances**
- Ví dụ: Firewalls, IDS/IPS, Deep Packet Inspection, payload manipulation
- Hoạt động ở **Layer 3 (Network Layer)** - IP Packets
- Kết hợp:
  - **Transparent Network Gateway** - single entry/exit cho tất cả traffic
  - **Load Balancer** - phân phối traffic đến virtual appliances
- Sử dụng **GENEVE protocol** on port 6081

**Target Groups:**
- EC2 instances
- IP Addresses (phải là private IPs)

### Sticky Sessions (Session Affinity) (Trang 111)

- Có thể implement stickiness để **cùng một client luôn được redirect đến cùng một instance**
- Hoạt động với **Classic LB, ALB và NLB**
- Cookie có **expiration date** mà bạn kiểm soát
- **Use case:** Đảm bảo user không bị mất session data
- **Cảnh báo:** Enabling stickiness có thể gây **imbalance** cho backend instances

### Cross-Zone Load Balancing (Trang 112-113)

**With Cross Zone Load Balancing:**
- Mỗi load balancer instance phân phối **đều nhau** across tất cả registered instances **trong tất cả AZ**

**Without Cross Zone Load Balancing:**
- Requests được phân phối **chỉ trong instances của node** đó

**Cấu hình:**
- **Application Load Balancer:** Enabled by default (có thể disable ở Target Group level), không charge cho inter AZ data
- **Network Load Balancer & Gateway Load Balancer:** Disabled by default, **có charge** cho inter AZ data nếu enabled

### SSL/TLS - Basics (Trang 114)

- **SSL Certificate** cho phép traffic giữa clients và load balancer được **encrypt in transit** (in-flight encryption)
- **SSL** = Secure Sockets Layer, dùng để encrypt connections
- **TLS** = Transport Layer Security, version mới hơn
- Ngày nay chủ yếu dùng **TLS certificates**, nhưng mọi người vẫn gọi là SSL
- **Public SSL certificates** được cấp bởi Certificate Authorities (CAs): Comodo, Symantec, GoDaddy, GlobalSign, Digicert, Letsencrypt
- SSL certificates có **expiration date** và phải được **renew**

### Load Balancer - SSL Certificates (Trang 115)

- Load balancer sử dụng **X.509 certificate**
- Quản lý certificates bằng **ACM (AWS Certificate Manager)**
- Có thể upload certificates của riêng bạn
- **HTTPS listener:**
  - Phải specify **default certificate**
  - Có thể add **optional list of certs** để hỗ trợ multiple domains
  - Clients có thể dùng **SNI (Server Name Indication)** để specify hostname

### SSL - Server Name Indication (SNI) (Trang 116)

- **SNI** giải quyết vấn đề loading **multiple SSL certificates** onto one web server
- **Lưu ý:**
  - Chỉ hoạt động với **ALB & NLB** (newer generation), CloudFront
  - **Không hoạt động** với CLB (older gen)

### Connection Draining / Deregistration Delay (Trang 117)

- **Feature naming:**
  - Connection Draining - cho CLB
  - Deregistration Delay - cho ALB & NLB

- Thời gian để complete **"in-flight requests"** trong khi instance đang de-registering hoặc unhealthy
- Ngừng gửi new requests đến EC2 instance đang de-registering
- Thời gian: **1 đến 3600 seconds** (default: 300 seconds)
- Có thể disable (set value = 0)
- Nên set **low value** nếu requests ngắn

---

## Phần 3: Auto Scaling Group - ASG (Trang 119-128)

### What's an Auto Scaling Group? (Trang 118-119)

- Trong thực tế, **load** trên websites và applications có thể thay đổi

**Mục tiêu của ASG:**
- **Scale out** (thêm EC2 instances) để match increased load
- **Scale in** (xóa EC2 instances) để match decreased load
- Đảm bảo có **minimum và maximum number** of EC2 instances running
- **Automatically register** new instances đến load balancer
- **Re-create** EC2 instance nếu instance trước bị terminated (ví dụ: nếu unhealthy)

> *Sơ đồ: ASG kết hợp với Load Balancer*

**Lưu ý:** ASG là **free** (chỉ trả tiền cho underlying EC2 instances)

### Auto Scaling Group Attributes (Trang 121)

**Launch Template** (Launch Configurations đã deprecated):
- AMI + Instance Type
- EC2 User Data
- EBS Volumes
- Security Groups
- SSH Key Pair
- IAM Roles cho EC2 Instances
- Network + Subnets Information
- Load Balancer Information
- Min Size / Max Size / Initial Capacity
- Scaling Policies

### Auto Scaling - CloudWatch Alarms & Scaling (Trang 122)

- Có thể scale ASG dựa trên **CloudWatch alarms**
- Alarm theo dõi một **metric** (ví dụ: Average CPU, custom metric)
- Metrics như **Average CPU** được tính cho **overall ASG instances**
- Dựa trên alarm:
  - Tạo **scale-out policies** (tăng số instances)
  - Tạo **scale-in policies** (giảm số instances)

### Auto Scaling Groups - Scaling Policies (Trang 123-124)

**1. Dynamic Scaling:**

- **Target Tracking Scaling** (strongly recommend)
  - Scale resource dựa trên **target value** cho một specific CloudWatch metric
  - Ví dụ: Giữ average ASG CPU ở khoảng 40%

- **Simple / Step Scaling**
  - Khi CloudWatch alarm triggered (CPU > 70%) → thêm 2 units
  - Khi CloudWatch alarm triggered (CPU < 30%) → xóa 1

- **Scheduled Scaling**
  - Scale dựa trên **known usage patterns**
  - Ví dụ: Tăng min capacity lên 10 vào 5pm thứ 6

**2. Predictive Scaling:**
- Sử dụng **machine learning** để predict capacity requirements dựa trên historical data từ CloudWatch

### Good Metrics to Scale On (Trang 125)

- **CPUUtilization**: Average CPU utilization across instances
- **RequestCountPerTarget**: Đảm bảo số requests per EC2 instance ổn định
- **Average Network In / Out** (nếu application bị network bound)
- **Custom metrics** (push từ CloudWatch)

### Auto Scaling Groups - Scaling Cooldowns (Trang 126)

- Sau khi scaling activity xảy ra, bạn ở trong **cooldown period** (default: 300 seconds)
- Trong cooldown period, ASG sẽ **không launch hoặc terminate** thêm instances
- Cho phép metrics **stabilize**

**Advice:** Sử dụng **ready-to-use AMI** để reduce configuration time, serve requests nhanh hơn và reduce cooldown period

### Auto Scaling - Instance Refresh (Trang 127)

- **Goal:** Update launch template và re-create tất cả EC2 instances
- Sử dụng native feature **Instance Refresh**
- Settings:
  - **Minimum healthy percentage**
  - **Warm-up time** (bao lâu cho đến khi instance ready to use)

---

## Phần 4: RDS, Aurora & ElastiCache (Trang 131-156)

### Amazon RDS Overview (Trang 131-132)

- **RDS** = Relational Database Service
- Managed DB service sử dụng **SQL** làm query language
- Tạo databases trong cloud được **quản lý bởi AWS**
- Hỗ trợ: PostgreSQL, MySQL, MariaDB, Oracle, Microsoft SQL Server, IBM DB2, Aurora

**Ưu điểm so với DB trên EC2:**
- **Automated provisioning, OS patching**
- **Continuous backups** và restore đến specific timestamp (Point in Time Restore)
- **Monitoring dashboards**
- **Read replicas** cho improved read performance
- **Multi AZ** setup cho DR (Disaster Recovery)
- **Maintenance windows** cho upgrades
- **Scaling capability** (vertical và horizontal)
- **Storage backed by EBS** (gp2 hoặc io1)

### RDS - Storage Auto Scaling (Trang 133)

- Giúp **tự động tăng storage** khi cần
- Khi RDS detect sắp hết free database storage, nó **scale tự động**
- Phải set **Maximum Storage Threshold**
- Tự động modify storage nếu:
  - Free storage < 10% of allocated storage
  - Low-storage kéo dài ít nhất 5 minutes
  - Đã 6 hours kể từ lần modify cuối
- Hữu ích cho **unpredictable workloads**
- Hỗ trợ **tất cả RDS database engines**

### RDS Read Replicas for Read Scalability (Trang 134-135)

- Tối đa **15 Read Replicas**
- Within AZ, Cross AZ hoặc Cross Region
- Replication là **ASYNC** → reads cuối cùng sẽ consistent
- Replicas có thể được **promoted** thành standalone DB
- Applications phải **update connection string** để leverage read replicas

**Use Cases:**
- Production database đang xử lý normal load
- Tạo Read Replica để **chạy reporting/analytics**
- Production application **không bị ảnh hưởng**
- Read replicas dùng cho **SELECT statements only** (không phải INSERT, UPDATE, DELETE)

**Lưu ý:** Trong cùng region, **không tốn phí** cho cross-AZ data transfer

### RDS Multi AZ (Disaster Recovery) (Trang 137-138)

- **SYNC replication**
- **Một DNS name** - automatic app failover to standby
- Tăng **availability**
- Failover khi:
  - Loss of AZ
  - Loss of network
  - Instance failure
  - Storage failure
- **Không cần manual intervention** trong apps
- **Không dùng cho scaling**

**Zero Downtime Migration từ Single-AZ sang Multi-AZ:**
1. Click "modify" cho database
2. Snapshot được taken
3. New DB được restore từ snapshot trong new AZ
4. Synchronization được established giữa hai databases

### Amazon Aurora (Trang 139-140)

- **Proprietary technology** từ AWS (không open sourced)
- Hỗ trợ cả **Postgres và MySQL**
- **"AWS cloud optimized"** - 5x performance improvement over MySQL on RDS, 3x over Postgres on RDS
- Aurora storage **tự động grow** in increments of 10GB, lên đến **128 TB**
- Có thể có **đến 15 replicas** với replication **nhanh hơn MySQL** (< 10ms replica lag)
- **Failover** gần như **instantaneous** - native HA
- **Chi phí cao hơn RDS** (20% more) nhưng **hiệu quả hơn**

### Aurora High Availability and Read Scaling (Trang 140-141)

- **6 copies** của data across **3 AZ**:
  - 4/6 copies cần thiết cho **writes**
  - 3/6 copies cần thiết cho **reads**
- **Self healing** với peer-to-peer replication
- Storage **striped across 100s of volumes**

**Architecture:**
- Một Aurora Instance làm **master** (writes)
- **Automated failover** cho master trong < 30 seconds
- Master + **đến 15 Aurora Read Replicas** serve reads
- Hỗ trợ **Cross Region Replication**

### Features of Aurora (Trang 142)

- Automatic fail-over
- Backup and Recovery
- Isolation and security
- Industry compliance
- Push-button scaling
- Automated Patching with Zero Downtime
- Advanced Monitoring
- Routine Maintenance
- **Backtrack**: restore data at any point without using backups

### RDS & Aurora Security (Trang 143)

**At-rest encryption:**
- Database master & replicas sử dụng **AWS KMS** - phải define at launch time
- Nếu master **không encrypted**, read replicas **không thể encrypted**
- Để encrypt un-encrypted database: snapshot → restore as encrypted

**In-flight encryption:**
- **TLS-ready by default**
- Sử dụng AWS TLS root certificates client-side

**Authentication & Access:**
- **IAM Authentication**: IAM roles để connect (thay vì username/password)
- **Security Groups**: Kiểm soát Network access
- **Không có SSH** (trừ RDS Custom)
- **Audit Logs** có thể enabled và gửi đến CloudWatch Logs

### Amazon RDS Proxy (Trang 144)

- **Fully managed database proxy** cho RDS
- Apps **pool và share DB connections**
- **Improving database efficiency** - giảm stress trên CPU, RAM, minimize open connections
- **Serverless**, autoscaling, highly available (multi-AZ)
- **Giảm failover time** lên đến 66%
- Hỗ trợ: RDS (MySQL, PostgreSQL, MariaDB, MS SQL Server) và Aurora (MySQL, PostgreSQL)
- **Không cần code changes** cho hầu hết apps
- Enforce **IAM Authentication**, lưu credentials trong **AWS Secrets Manager**
- RDS Proxy **không public** - phải access từ VPC

### Amazon ElastiCache Overview (Trang 145-146)

- Giống như RDS cho managed Relational Databases...
- **ElastiCache** cho managed **Redis hoặc Memcached**
- Caches là **in-memory databases** với **high performance, low latency**
- Giúp **giảm load** trên databases cho **read intensive workloads**
- Giúp applications **stateless**
- AWS lo: OS maintenance/patching, optimizations, setup, configuration, monitoring, failure recovery, backups
- **Cần thay đổi code nhiều** khi sử dụng ElastiCache

### ElastiCache Solution Architecture (Trang 146-147)

**1. DB Cache Pattern:**
- Application queries ElastiCache → nếu không có → get từ RDS → store in ElastiCache
- Giúp **relieve load trong RDS**
- Cần có **invalidation strategy** để đảm bảo chỉ data mới nhất được dùng

**2. User Session Store Pattern:**
- User login vào bất kỳ application instance nào
- Application viết session data vào ElastiCache
- User hit another instance → instance retrieve data → user đã logged in

### ElastiCache - Redis vs Memcached (Trang 148)

| Feature | **Redis** | **Memcached** |
|---------|-----------|---------------|
| **Replication** | Multi AZ with Auto-Failover | Sharding (Multi-node) |
| **High Availability** | Có (Read Replicas) | Không |
| **Data Durability** | AOF persistence | Non-persistent |
| **Backup & Restore** | Có | Không |
| **Data Types** | Sets, Sorted Sets | Simple key-value |
| **Architecture** | Replication | Sharding |
| **Threading** | Single-threaded | Multi-threaded |

### Caching Implementation Considerations (Trang 149)

**Câu hỏi cần đặt ra:**
- **Is it safe to cache data?** Data có thể outdated, eventually consistent
- **Is caching effective?** Pattern: data thay đổi chậm, vài keys được request thường xuyên
- **Is data structured well?** Ví dụ: key-value caching, caching aggregations

**Anti-patterns:**
- Data thay đổi nhanh
- All large key space được request thường xuyên

### Lazy Loading / Cache-Aside (Trang 150-151)

> *Sơ đồ: Cache-Aside Pattern Flow*

**Pros:**
- Chỉ **requested data được cached** (cache không bị fill với unused data)
- **Node failures không fatal** (chỉ tăng latency để warm cache)

**Cons:**
- **Cache miss penalty** → 3 round trips, noticeable delay
- **Stale data**: Data có thể updated trong DB nhưng outdated trong cache

### Write Through (Trang 152-153)

- **Add hoặc Update cache** khi database được updated

**Pros:**
- Data trong cache **không bao giờ stale**
- Reads nhanh

**Cons:**
- **Missing data** cho đến khi added/updated in DB
- **Cache churn** - nhiều data không bao giờ được read
- **Write penalty** (mỗi write cần 2 calls)

### Cache Evictions and Time-to-Live (TTL) (Trang 154)

**Cache eviction xảy ra theo 3 cách:**
1. **Explicitly delete** item trong cache
2. **Evicted** vì memory full và không recently used (**LRU**)
3. **Set TTL** cho item

**TTL hữu ích cho:**
- Leaderboards
- Comments
- Activity streams

- TTL có thể từ **few seconds đến hours hoặc days**
- Nếu quá nhiều evictions vì memory full → **scale up hoặc out**

### Final Words on Caching (Trang 155)

- **Lazy Loading** dễ implement và hoạt động cho nhiều situations, đặc biệt cho **read side**
- **Write-through** thường được kết hợp với Lazy Loading cho queries/workloads cần optimization
- **Setting TTL** thường không phải ý tưởng tồi, trừ khi dùng Write-through
- **Chỉ cache data hợp lý** (user profiles, blogs, etc.)

---

## Phần 5: Amazon Route 53 (Trang 157-160+)

### What is DNS? (Trang 157-158)

- **DNS** = Domain Name System - translate human-friendly hostnames thành machine IP addresses
- Ví dụ: `www.google.com` → `172.217.18.36`
- DNS là **backbone của Internet**
- DNS sử dụng **hierarchical naming structure**

> *Sơ đồ: .com → example.com → www.example.com → api.example.com*

### DNS Terminologies (Trang 158-159)

| Term | Definition |
|------|------------|
| **Domain Registrar** | Amazon Route 53, GoDaddy, ... |
| **DNS Records** | A, AAAA, CNAME, NS, ... |
| **Zone File** | Chứa DNS records |
| **Name Server** | Resolves DNS queries (Authoritative or Non-Authoritative) |
| **Top Level Domain (TLD)** | .com, .us, .in, .gov, .org, ... |
| **Second Level Domain (SLD)** | amazon.com, google.com, ... |

### How DNS Works (Trang 159-160)

> *Sơ đồ: Client → Resolver → Root Server → TLD Server → Authoritative Server*
