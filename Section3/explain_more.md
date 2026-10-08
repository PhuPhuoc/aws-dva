# Section 3 - Extended Knowledge: ELB, ASG, RDS, Aurora, ElastiCache, Route 53

---

## 1. Elastic Load Balancer (ELB)

### ALB vs NLB vs CLB - Khi nào nên dùng?

**DO (NÊN):**
- Sử dụng **Application Load Balancer (ALB)** cho hầu hết web applications vì:
  - Hỗ trợ **Layer 7 routing** (HTTP/HTTPS)
  - **Path-based routing** và **host-based routing**
  - Tích hợp tốt với **microservices và containers**
  - Hỗ trợ **Lambda** như target
  - **Fixed hostname** dễ quản lý

- Sử dụng **Network Load Balancer (NLB)** khi:
  - Cần **extreme performance** (millions requests/second)
  - Cần **low latency** (~100ms vs 400ms của ALB)
  - Cần **static IP per AZ** hoặc **Elastic IP**
  - Cần forward **TCP/UDP traffic**

- Sử dụng **Gateway Load Balancer** khi:
  - Cần deploy **firewalls, IDS/IPS**
  - Cần **transparent network gateway**
  - Cần inspect/modify traffic qua virtual appliances

**DON'T (KHÔNG NÊN):**
- Không dùng **Classic Load Balancer (CLB)** cho các application mới - đã deprecated
- Không dùng NLB khi chỉ cần HTTP/HTTPS routing vì:
  - Không hỗ trợ **path-based routing**
  - Không hỗ trợ **host-based routing**
  - Không tích hợp tốt với containers/microservices
  - Chi phí cao hơn ALB

### Security Groups Configuration Best Practices

**DO (NÊN):**
```
Load Balancer Security Group:
- Inbound: 0.0.0.0/0 (HTTP/HTTPS)
- Outbound: 0.0.0.0/0

Application/EC2 Security Group:
- Inbound: Chỉ từ Load Balancer SG (security group reference)
- Outbound: 0.0.0.0/0
```

**Lợi ích của Security Group referencing:**
- Khi thêm instance mới, tự động được allow
- Không cần hardcode IP addresses
- Tự động cập nhật khi instances thay đổi

**DON'T (KHÔNG NÊN):**
- Không allow traffic trực tiếp từ Internet đến EC2
- Không hardcode IP addresses trong rules
- Không mở port 22 (SSH) ra Internet

### SSL/TLS Certificates

**DO (NÊN):**
- Luôn sử dụng **ACM (AWS Certificate Manager)** để quản lý certificates
- Enable **HTTPS** trên Load Balancer
- Sử dụng **SNI** để host multiple domains trên cùng ALB/NLB
- Rotate certificates trước khi expired

**DON'T (KHÔNG NÊN):**
- Không upload self-signed certificates cho production
- Không ignore certificate expiration
- Không dùng CLB nếu cần SNI (không hỗ trợ)

**💡 Tip: X-Forwarded Headers**
```
X-Forwarded-For: Client IP
X-Forwarded-Port: Client port
X-Forwarded-Proto: HTTP/HTTPS
```
Luôn log X-Forwarded-For để track client IPs vì instances không thấy trực tiếp client IP.

### Cross-Zone Load Balancing

**DO (NÊN):**
- Với ALB: **Enable by default** - không cần change
- Với NLB: Consider enable nếu cần distribute traffic đều across all AZs

**⚠️ NOTE:**
- Với NLB, khi enable Cross-Zone LB, sẽ **tính phí** inter-AZ data transfer
- Tính toán chi phí trước khi enable cho NLB

### Connection Draining / Deregistration Delay

**DO (NÊN):**
- Set **30-60 seconds** cho general web applications
- Set **5-10 seconds** cho applications với short requests
- Set **300+ seconds** cho applications với long-running requests (file uploads)

**DON'T (KHÔNG NÊN):**
- Set quá ngắn cho applications với long requests
- Disable completely trừ khi có lý do cụ thể

---

## 2. Auto Scaling Group (ASG)

### Launch Template vs Launch Configuration

**DO (NÊN):**
- Luôn dùng **Launch Template** (Launch Configurations đã deprecated)
- Version control cho Launch Template
- Sử dụng **latest version** hoặc specify version cụ thể

**💡 Tip: User Data Optimization**
```bash
#!/bin/bash
# Sử dụng pre-baked AMI thay vì bootstrap
# Ví dụ: Codepipeline agent đã install sẵn
# Giảm bootstrap time từ 5-10 phút xuống 30 giây
```

### Scaling Policies Best Practices

**DO (NÊN):**
- Sử dụng **Target Tracking Scaling** làm primary policy
- Set **multiple metrics** để scale (CPU + RequestCountPerTarget)
- Implement **scale-in policies** cùng với scale-out
- Set **appropriate cooldown** (300-600 seconds thường OK)

**Recommended Target Tracking Configurations:**
| Metric | Target Value | Use Case |
|--------|-------------|----------|
| ASGAverageCPUUtilization | 40-50% | General web apps |
| ALBRequestCountPerTarget | 100-500 | Load-balanced apps |
| RDSReaderAverageCPUUtilization | 70% | Database reads |
| ElastiCacheDatabaseMemoryUsagePercentage | 70-80% | Cache optimization |

**DON'T (KHÔNG NÊN):**
- Không scale out quá nhanh (step scaling với large increments)
- Không set target quá thấp (ví dụ: CPU 20%) - tốn chi phí
- Không ignore scale-in policies
- Không set cooldown quá ngắn (dưới 60s)

### Scheduled Scaling

**DO (NÊN):**
- Use for **predictable traffic patterns**
- Scale up trước khi peak hours
- Scale down sau khi peak hours kết thúc

**Ví dụ:**
```json
{
  "ScheduledAction": "Scale-Out-Before-Peak",
  "Schedule": "0 17 * * 5",  // 5PM every Friday
  "MinSize": 10,
  "DesiredCapacity": 10
}
```

### Instance Refresh

**DO (NÊN):**
- Always set **minimum healthy percentage** (recommend 90%)
- Set **warm-up time** để instance ready trước khi receive traffic
- Test in staging environment trước

**DON'T (KHÔNG NÊN):**
- Không run instance refresh trong production peak hours
- Không set minimum healthy percentage = 0

---

## 3. RDS (Relational Database Service)

### Multi-AZ vs Read Replicas

**DO (NÊN):**
- Enable **Multi-AZ** cho production databases
- Sử dụng **Read Replicas** để:
  - Scale read-heavy workloads
  - Run analytics/reporting queries
  - Perform backups without impacting primary

**DON'T (KHÔNG NÊN):**
- Không coi Multi-AZ là scaling solution (nó chỉ cho HA)
- Không sử dụng Read Replicas cho write operations
- Không ignore replication lag khi app sensitive to stale data

**⚠️ IMPORTANT:**
```
Multi-AZ: SYNC replication, automatic failover, same region (different AZ)
Read Replicas: ASYNC replication, manual promotion, can be cross-region
```

### RDS Storage

**DO (NÊN):**
- Enable **Storage Auto Scaling** nếu workload unpredictable
- Set **Maximum Storage Threshold** cao hơn expected peak
- Monitor **FreeStorageSpace** metrics

**💡 Tip: Storage Types**
- **gp3**: 20% cheaper than gp2, baseline 3000 IOPS
- **gp2**: Burstable, good for dev/test
- **io1/io2**: For high IOPS requirements (need Provisioned IOPS)

### RDS Security

**DO (NÊN):**
- Enable **Encryption at Rest** (KMS)
- Enforce **SSL/TLS** for connections
- Use **IAM Authentication** thay vì password
- Store credentials trong **Secrets Manager**
- Sử dụng **VPC Security Groups** để restrict access

**DON'T (KHÔNG NÊN):**
- Không public accessible RDS instances
- Không dùng default credentials
- Không ignore **Parameter Groups** settings

### RDS Proxy

**DO (NÊN):**
- Sử dụng RDS Proxy khi:
  - Many short-lived connections
  - Serverless applications
  - Lambda functions connecting to RDS
  - Cần reduce **failover time**
- Enable IAM authentication qua RDS Proxy

**DON'T (KHÔNG NÊN):**
- Không dùng khi cần long-running queries (> 60 seconds)
- Không dùng với transactions spanning multiple connections

---

## 4. Amazon Aurora

### Aurora vs RDS

**DO (NÊN):**
- Chọn **Aurora** khi cần:
  - **High performance** (5x MySQL, 3x Postgres)
  - **Auto-scaling storage** (10GB to 128TB)
  - **Fast failover** (< 30 seconds)
  - **Cross-region replication**
  - **Backtrack feature** (restore without backups)

- Chọn **RDS** khi:
  - Đã có existing MySQL/Postgres apps
  - Cần **simpler management**
  - Budget constraints (Aurora đắt hơn 20%)

**Aurora Serverless:**
- Good cho **intermittent/unpredictable workloads**
- Auto-pause khi idle
- Good cho dev/test environments

**⚠️ NOTE: Aurora Storage**
- Storage auto-scales từ 10GB increments
- Không cần provision storage upfront
- Tính phí theo actual usage

### Aurora Replicas

**DO (NÊN):**
- Tạo **Aurora Replicas** cho:
  - Read scaling
  - HA (automatic failover)
  - Geographic distribution (Cross-region)

**⚠️ IMPORTANT:**
- Aurora replicas ** không có replication lag đáng kể** (< 10ms)
- Replicas có thể be promoted **instantly**
- Tất cả replicas có thể serve reads

---

## 5. Amazon ElastiCache

### Redis vs Memcached

**DO (NÊN):**
- Chọn **Redis** khi cần:
  - **High availability** (replication, failover)
  - **Data persistence** (AOF, RDB)
  - **Advanced data types** (Sets, Sorted Sets, Streams)
  - **Pub/Sub** capabilities
  - **Lua scripting**

- Chọn **Memcached** khi cần:
  - **Simple key-value caching**
  - **Multi-threaded** performance
  - **Horizontal scaling** (sharding)
  - **Lowest cost** per node

**💡 Tip: Use Cases**
| Use Case | Recommended |
|----------|-------------|
| Session store | Redis |
| Caching query results | Redis or Memcached |
| Real-time leaderboards | Redis (Sorted Sets) |
| Chat/messaging | Redis (Pub/Sub) |
| Rate limiting | Redis |
| Simple HTML caching | Memcached |

### Caching Strategies

**DO (NÊN):**

**1. Cache-Aside (Lazy Loading):**
```python
def get_user(user_id):
    # 1. Check cache first
    user = cache.get(f"user:{user_id}")
    if user:
        return user
    
    # 2. Cache miss - get from DB
    user = db.query("SELECT * FROM users WHERE id = ?", user_id)
    
    # 3. Store in cache with TTL
    cache.set(f"user:{user_id}", user, ttl=3600)
    return user
```

**2. Write-Through:**
```python
def update_user(user_id, data):
    # 1. Write to DB
    db.execute("UPDATE users SET ... WHERE id = ?", user_id, data)
    
    # 2. Update cache immediately
    user = db.query("SELECT * FROM users WHERE id = ?", user_id)
    cache.set(f"user:{user_id}", user, ttl=3600)
```

**When to use which:**
| Strategy | Best For | Avoid When |
|----------|----------|------------|
| Cache-Aside | Read-heavy, tolerate stale data | Write-heavy, data must be fresh |
| Write-Through | Data must be fresh, predictable writes | Write-heavy, many keys never read |

**DON'T (KHÔNG NÊN):**
- Không cache **entire database tables**
- Không set **TTL quá dài** cho frequently changing data
- Không ignore **cache invalidation strategy**
- Không dùng cache như primary data store

### TTL Best Practices

**DO (NÊN):**
- Set TTL based on data freshness requirements:
  - **Session data**: 15-30 minutes
  - **User profiles**: 1-24 hours
  - **Product catalog**: 1-24 hours
  - **Real-time data**: No cache / very short TTL

**⚠️ NOTE: TTL Anti-patterns**
- TTL = 0: Item never expires (khiến cache stale)
- TTL = infinite: Tương tự, cache forever
- Always set explicit TTL

### Cache Eviction

**DO (NÊN):**
- Monitor **CacheHitRate** và **CurrItems**
- Scale up (increase node size) khi memory pressure
- Scale out (add nodes) khi need more throughput

**⚠️ IMPORTANT:**
- Redis eviction policies: `allkeys-lru`, `volatile-lru`, `noeviction`
- Set appropriate policy based on use case
- Monitor **evictions** metric

---

## 6. Route 53

### DNS Records Types

**DO (NÊN):**
- **A Record**: Map domain to IPv4
- **AAAA Record**: Map domain to IPv6
- **CNAME**: Map subdomain to another domain
- **Alias Record**: Map domain to AWS resource (ALB, S3, CloudFront)

**⚠️ IMPORTANT: CNAME vs Alias**
- CNAME: Chỉ cho non-root domains (www.example.com)
- Alias: Cho cả root domains (example.com)
- Alias: **Không charge** cho queries
- Alias: Tự động resolve IP changes

### Routing Policies

**DO (NÊN):**

| Policy | Use Case |
|--------|----------|
| **Simple** | Single resource |
| **Weighted** | A/B testing, blue-green deployments |
| **Latency-based** | Lowest latency for users |
| **Failover** | Active-passive HA |
| **Geolocation** | Regional content, compliance |
| **Geoproximity** | Traffic flow based on geography |
| **IP-based** | Custom routing based on IP ranges |
| **Multi-value** | Multiple healthy resources |

**Weighted Routing Example:**
```
- version-1.example.com: Weight 90 (90% traffic)
- version-2.example.com: Weight 10 (10% traffic)
```

### Health Checks

**DO (NÊN):**
- Enable **Health Checks** for all critical resources
- Use **calculated health checks** for composite monitoring
- Set appropriate **thresholds** (consecutive failures)

**⚠️ IMPORTANT:**
- Health checks check every 10/30 seconds (fastest)
- 3xx responses are considered healthy
- String matching requires response to contain string

**DON'T (KHÔNG NÊN):**
- Không disable health checks trong production
- Không point health check trực tiếp đến IP (dùng DNS name)

### Route 53 Resolver

**DO (NÊN):**
- Sử dụng **Resolver Endpoints** cho hybrid DNS
- Inbound Endpoints: Cho on-premises queries Route 53
- Outbound Endpoints: Cho Route 53 queries on-premises

---

## 7. Architecture Patterns Summary

### High Availability Architecture

```
                    ┌─────────────────┐
                    │   Route 53       │
                    │  (Failover)     │
                    └────────┬────────┘
                             │
              ┌──────────────┴──────────────┐
              │                             │
       ┌──────▼──────┐              ┌──────▼──────┐
       │    ALB       │              │    ALB       │
       │   (AZ 1)     │              │   (AZ 2)     │
       └──────┬──────┘              └──────┬──────┘
              │                             │
       ┌──────▼──────┐              ┌──────▼──────┐
       │    ASG       │              │    ASG       │
       │ (Multi-AZ)   │              │ (Multi-AZ)   │
       └──────┬──────┘              └──────┬──────┘
              │                             │
       ┌──────▼──────┐              ┌──────▼──────┐
       │  RDS Master │ ───Replica──▶│ RDS Standby │
       │   (AZ 1)    │              │   (AZ 2)    │
       └─────────────┘              └─────────────┘
```

### Read-Heavy Architecture

```
                    ┌─────────────────┐
                    │   Load Balancer  │
                    └────────┬────────┘
                             │
              ┌──────────────┴──────────────┐
              │                             │
       ┌──────▼──────┐              ┌──────▼──────┐
       │   EC2/Win    │              │   EC2/Win   │
       │  Application │              │  Application │
       └──────┬──────┘              └──────┬──────┘
              │                             │
       ┌──────▼──────────────────────────────▼──────┐
       │              ElastiCache (Redis)            │
       │                  (Read Replicas)             │
       └───────────────────────┬──────────────────────┘
                             │
       ┌─────────────────────┴─────────────────────┐
       │                  RDS Aurora                 │
       │           (Master + 2 Read Replicas)       │
       └────────────────────────────────────────────┘
```

---

## Key Differences Summary Table

| Feature | ALB | NLB | CLB |
|---------|-----|-----|-----|
| Layer | 7 (HTTP/HTTPS) | 4 (TCP/UDP) | 4/7 |
| Path Routing | ✅ | ❌ | ❌ |
| Host Routing | ✅ | ❌ | ❌ |
| SNI | ✅ | ✅ | ❌ |
| Static IP | ❌ | ✅ | ❌ |
| Zonal isolation | Per AZ | Per AZ | Cross-AZ |
| Free Tier | ✅ | ❌ | ✅ (deprecated) |

---

## Exam Tips

**High-Yield Topics:**
1. ALB routing features (path, host, query string)
2. ASG scaling policies (target tracking, scheduled)
3. RDS Multi-AZ vs Read Replicas
4. Aurora vs RDS differences
5. ElastiCache strategies (lazy loading vs write-through)
6. Route 53 routing policies
7. SSL/TLS with ALB/NLB (SNI)

**Common Exam Scenarios:**
- "Need to route traffic based on URL path" → ALB
- "Need to handle millions of requests with low latency" → NLB
- "Need to improve read performance" → Read Replicas
- "Need to cache database queries" → ElastiCache
- "Need zero-downtime database migration" → Multi-AZ
- "Need to test new version with small traffic" → Weighted routing
