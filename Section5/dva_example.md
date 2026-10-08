# Section 5 - DVA Practice Questions: VPC Advanced, ECS, ECR, Fargate, EKS, CloudFront

---

## 1. VPC Advanced

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

**Why C is correct:**
- **Bastion Host** là EC2 instance trong public subnet
- Cho phép SSH vào private instances từ Internet
- Bastion Host phải:
  - Nằm trong **public subnet**
  - Allow inbound SSH từ Internet (port 22)
  - Private instances allow SSH từ Bastion Security Group

**Architecture:**
```
Internet → Bastion Host (Public) → Private EC2 Instances
```

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. NAT Gateway | Chỉ cho phép outbound traffic, không phải SSH access |
| B. IGW | Cần EC2 có public IP, không cho phép SSH trực tiếp |
| D. VPC Peering | Kết nối VPCs, không phải SSH access |

### 📝 Vocabulary Analysis:
- **Bastion Host**: Jump server trong public subnet
- **Jump server**: Server để access vào private network

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

**Why B is correct:**

| Feature | Security Group | NACL |
|---------|---------------|------|
| **Level** | Instance | Subnet |
| **State** | **Stateful** | **Stateless** |
| **Rules** | Allow only | Allow + Deny |
| **Evaluation** | All rules | By order |
| **Return traffic** | Auto-allowed | Must explicitly allow |

**Stateful vs Stateless:**
- **Stateful**: Nếu allow inbound, return traffic auto-allowed
- **Stateless**: Phải allow cả outbound traffic cho return traffic

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A | Ngược lại - Security Groups là stateful |
| C, D | Một cái stateful, một stateless |

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

**Why C is correct:**
- **VPC Gateway Endpoint** cho phép:
  - Private access đến S3/DynamoDB
  - Traffic không qua Internet
  - **Free** (không tốn phí)
  - Sử dụng route table

**Two types of VPC Endpoints:**

| Type | Services | Cost | Uses |
|------|----------|------|------|
| **Gateway** | S3, DynamoDB | Free | Route table |
| **Interface** | Other services | $ per hour | ENI + SG |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. NAT Gateway | Cho phép Internet access, không phải S3 access |
| B. IGW | Traffic qua Internet, không phải private access |
| D. VPC Peering | Kết nối VPCs, không phải AWS services |

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

**Why B is correct:**
- **NACLs** supports **DENY rules**
- Applied ở **subnet level**
- Rules evaluated by order (lowest number first)
- Perfect để block specific IP addresses

**Example NACL:**
```
Rule #: 100 - Allow HTTP (80)
Rule #: 200 - Deny specific IP
Rule #: *  - Deny all
```

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Security Group | Chỉ supports ALLOW rules, không có DENY |
| C. VPC Peering | Kết nối VPCs, không block IPs |
| D. Route Table | Định hướng traffic, không block IPs |

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

**Why C is correct:**
- **NAT Gateway** specs:
  - Starts at **5 Gbps**
  - **Auto-scales** up to **100 Gbps**
  - **No Security Groups** needed
  - AWS managed, high availability
  - Pay per hour + bandwidth

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A | Too low - NAT Gateway has higher bandwidth |
| B | Correct bandwidth, wrong max (45 Gbps) |
| D | Wrong - doesn't scale to 10 Gbps |

---

## 2. ECS Fundamentals

### Question 6:
What is the basic unit of work in ECS?

```
A. Cluster
B. Service
C. Task Definition
D. Task
```

### ✅ Answer: **D. Task**

### 📖 Explanation (Vietnamese):

**Why D is correct:**
- **Task** = basic unit of work in ECS
- Task contains **one or more containers**
- Running container với settings từ Task Definition
- Task Definition = metadata (JSON) describing how to run

**ECS Hierarchy:**
```
Cluster
  └── Service (keeps tasks running)
        └── Task Definition (how to run)
              └── Task (running instance)
```

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Cluster | Group of services and tasks |
| B. Service | Supervises tasks, keeps them running |
| C. Task Definition | Blueprint/metadata for tasks |

---

### Question 7:
In ECS, which IAM role is used by the ECS agent on EC2 instances?

```
A. ECS Task Role
B. EC2 Instance Profile
C. Service Role
D. Task Execution Role
```

### ✅ Answer: **B. EC2 Instance Profile**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **EC2 Instance Profile** used by ECS agent:
  - Makes API calls to ECS service
  - Send container logs to CloudWatch
  - Pull Docker images from ECR
  - Access Secrets Manager/SSM Parameter Store

**Two types of ECS IAM Roles:**

| Role | Who Uses | Purpose |
|------|---------|---------|
| **EC2 Instance Profile** | ECS Agent (EC2 Launch Type) | Agent operations |
| **ECS Task Role** | Tasks themselves | Task permissions |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. ECS Task Role | Cho permissions của task, không phải agent |
| C. Service Role | Cho ECS service, không phải agent |
| D. Task Execution Role | Cho pulling images, không phải agent |

---

### Question 8:
What are the two launch types supported by ECS?

```
A. EC2 and Fargate
B. EC2 and Lambda
C. Fargate and Lambda
D. EC2 and EKS
```

### ✅ Answer: **A. EC2 and Fargate**

### 📖 Explanation (Vietnamese):

**Why A is correct:**
- **ECS supports two launch types:**
  - **EC2 Launch Type**: You manage EC2 instances
  - **Fargate Launch Type**: AWS manages infrastructure (Serverless)

**Comparison:**

| Feature | EC2 Launch Type | Fargate Launch Type |
|---------|----------------|-------------------|
| Infrastructure | You manage | AWS manages |
| Scaling | Manual + ASG | Automatic |
| Cost | EC2 instance hours | Per-task |
| Security Groups | Per instance | Per task (awsvpc) |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| B, C. Lambda | Lambda là serverless compute khác, không phải ECS launch type |
| D. EKS | EKS là Kubernetes service, không phải ECS launch type |

---

## 3. ECS Fargate

### Question 9:
Which ECS feature allows tasks to share data between multiple containers in the same task?

```
A. EFS volumes
B. Bind Mounts
C. EBS volumes
D. S3 buckets
```

### ✅ Answer: **B. Bind Mounts**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Bind Mounts**:
  - Share data between containers in same task
  - EC2: Uses EC2 instance storage
  - Fargate: Uses ephemeral storage (20-200 GiB)
  - Good for **sidecar pattern** (logs/metrics)

**Volume types comparison:**

| Type | Storage | Use Case |
|------|---------|----------|
| **Bind Mounts** | Host/Ephemeral | Shared data, sidecar |
| **EFS** | Network file system | Persistent, multi-AZ |
| **EBS** | Block storage | Single instance |
| **S3** | Object storage | Not mountable |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. EFS | Persistent shared storage, not between containers in same task |
| C. EBS | Block storage, not for container sharing |
| D. S3 | Object storage, cannot be mounted |

---

### Question 10:
A company wants to run containers without managing any EC2 instances. Which ECS launch type should be used?

```
A. EC2 Launch Type
B. Fargate Launch Type
C. Both EC2 and Fargate
D. EKS with Fargate
```

### ✅ Answer: **B. Fargate Launch Type**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Fargate Launch Type**:
  - **No EC2 instances** to manage
  - **Serverless** - AWS handles infrastructure
  - Just specify **CPU and memory**
  - AWS runs containers based on requirements
  - Scale by increasing **number of tasks**

**Fargate characteristics:**
- No provisioning
- No cluster management
- Pay per task (vCPU + memory used)
- Good for microservices

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. EC2 | Requires managing EC2 instances |
| C. Both | Doesn't match "without managing any EC2" |
| D. EKS | EKS là Kubernetes, không phải ECS |

---

## 4. ECR

### Question 11:
A developer needs to push a Docker image to Amazon ECR. Which command authenticates Docker to ECR?

```
A. aws ecr login
B. aws ecr get-login-password | docker login
C. docker login -u AWS
D. aws ecr authorize
```

### ✅ Answer: **B. aws ecr get-login-password | docker login**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **ECR Docker login command (AWS CLI v2):**
```bash
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  123456789.dkr.ecr.us-east-1.amazonaws.com
```

**Steps:**
1. Get ECR password via AWS CLI
2. Pipe to docker login
3. Docker authenticates with ECR

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. aws ecr login | Không tồn tại command này |
| C. docker login -u AWS | Không đúng format |
| D. aws ecr authorize | Không tồn tại command này |

### 📝 Vocabulary Analysis:
- **Docker login**: Authenticate Docker client to registry
- **ECR endpoint**: Account-specific registry URL

---

### Question 12:
Which storage backend does Amazon ECR use?

```
A. Amazon EBS
B. Amazon S3
C. Amazon EFS
D. Amazon DynamoDB
```

### ✅ Answer: **B. Amazon S3**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Amazon ECR is backed by Amazon S3**
- S3 provides durable storage for container images
- Combined with other AWS services for reliability
- ECR stores:
  - Image layers
  - Manifests
  - Metadata

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. EBS | Block storage, không phải cho container registry |
| C. EFS | File system, không phải cho images |
| D. DynamoDB | NoSQL database, không phải image storage |

---

## 5. CloudFront

### Question 13:
What is the main benefit of using CloudFront?

```
A. Reduces EC2 costs
B. Improves read performance by caching content at edge locations
C. Provides database storage
D. Manages container deployments
```

### ✅ Answer: **B. Improves read performance by caching content at edge locations**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **CloudFront** là CDN (Content Delivery Network)
- **Benefits:**
  - Content cached at **600+ edge locations** globally
  - Improved **read performance**
  - Reduced latency for users
  - DDoS protection
  - Integration with **Shield, WAF**

**How it works:**
```
User → Edge Location → Origin (S3/ALB)
           ↓
       Cached content (for subsequent requests)
```

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Reduces EC2 costs | Không phải primary benefit |
| C. Database storage | CloudFront không cung cấp storage |
| D. Container deployments | ECS/EKS làm việc này |

---

### Question 14:
Which CloudFront feature allows traffic to failover to a secondary origin when the primary fails?

```
A. Cache Behaviors
B. Origin Groups
C. Field-Level Encryption
D. Lambda@Edge
```

### ✅ Answer: **B. Origin Groups**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Origin Groups** provide:
  - **High availability**
  - **Failover** capability
  - One **primary** and one **secondary** origin
  - Auto failover when primary fails

**Use case:**
- S3 bucket + CloudFront (primary)
- S3 bucket in another region (secondary)
- Automatic failover if primary region unavailable

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Cache Behaviors | Different settings for URL patterns |
| C. Field-Level Encryption | Encrypt data at field level |
| D. Lambda@Edge | Run Lambda at edge locations |

---

### Question 15:
When origin content is updated, but CloudFront still serves old cached content. How can you force cache refresh?

```
A. Update TTL
B. Create CloudFront Invalidation
C. Delete CloudFront distribution
D. Update Security Groups
```

### ✅ Answer: **B. Create CloudFront Invalidation**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **CloudFront Invalidation**:
  - Forces cache refresh
  - Bypasses TTL
  - Can invalidate specific paths or all files

**CLI command:**
```bash
# Invalidate specific path
aws cloudfront create-invalidation \
  --distribution-id EDFDVBD6EXAMPLE \
  --paths "/images/*"

# Invalidate everything
aws cloudfront create-invalidation \
  --distribution-id EDFDVBD6EXAMPLE \
  --paths "/*"
```

**Note:** Invalidation takes time; charged for many invalidations

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Update TTL | TTL chỉ expiry sau thời gian, không immediate |
| C. Delete distribution | Extreme, unnecessary |
| D. Security Groups | Không liên quan đến caching |

---

### Question 16:
Which type of CloudFront origin requires enabling static website hosting on S3?

```
A. S3 bucket
B. S3 bucket as static website endpoint
C. Application Load Balancer
D. EC2 instance
```

### ✅ Answer: **B. S3 bucket as static website endpoint**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **S3 static website hosting**:
  - Must enable on S3 bucket
  - Uses website endpoint format
  - Requires bucket policy for public access
  - Block Public Access must be OFF

**S3 endpoints comparison:**

| Origin Type | Endpoint | Use Case |
|------------|----------|----------|
| S3 bucket (REST) | `s3.amazonaws.com` | General |
| S3 website | `s3-website.region.amazonaws.com` | Static website |

**Configuration needed:**
1. Enable Static Website Hosting
2. Set index document
3. Block Public Access: OFF
4. Bucket policy (public read)

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. S3 bucket (REST) | Không cần static website hosting |
| C. ALB | Custom origin HTTP, không phải S3 |
| D. EC2 | Custom origin HTTP, không phải S3 |

---

## 6. EKS

### Question 17:
Which AWS service is used to launch managed Kubernetes clusters on AWS?

```
A. Amazon ECS
B. Amazon EKS
C. Amazon ECR
D. AWS Fargate
```

### ✅ Answer: **B. Amazon EKS**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **EKS** = Elastic Kubernetes Service
- Launch managed **Kubernetes clusters** on AWS
- **Kubernetes**:
  - Open-source system
  - Automatic deployment, scaling
  - Management of containerized apps
- Alternative to ECS, **same goal, different API**

**AWS Container Services:**

| Service | Description |
|---------|-------------|
| **ECS** | AWS's own container platform |
| **EKS** | Managed Kubernetes |
| **Fargate** | Serverless containers |
| **ECR** | Container image registry |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. ECS | AWS's own platform, không phải Kubernetes |
| C. ECR | Container registry, không phải orchestration |
| D. Fargate | Serverless compute, không phải Kubernetes |

---

## 7. Combined Scenarios

### Question 18:
A company needs to process messages from an SQS queue using containers. Which ECS feature allows tasks to be triggered by SQS?

```
A. ECS Service
B. ECS Task Definition
C. EventBridge Scheduler
D. ECS Task invoked by EventBridge
```

### ✅ Answer: **D. ECS Task invoked by EventBridge**

### 📖 Explanation (Vietnamese):

**Why D is correct:**
- **EventBridge** can trigger ECS tasks:
  - SQS → EventBridge → ECS Task
  - Configure EventBridge rule
  - Target = ECS task
  - Supports:
    - Scheduled tasks
    - Event-based triggers (S3, SQS, etc.)

**Architecture:**
```
SQS Queue
    ↓
EventBridge Rule
    ↓
ECS Task (runs, processes, exits)
```

**Use cases:**
- Process SQS messages
- Batch jobs
- Scheduled workloads

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. ECS Service | Long-running tasks, không phải event-triggered |
| B. Task Definition | Blueprint, không trigger tasks |
| C. EventBridge Scheduler | Chỉ scheduled, không event-based |

---

### Question 19:
An application needs to store sensitive configuration data (API keys) and make it available to ECS tasks. Which service should be used?

```
A. Amazon S3
B. Amazon DynamoDB
C. AWS Secrets Manager
D. Amazon CloudWatch
```

### ✅ Answer: **C. AWS Secrets Manager**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **Secrets Manager**:
  - Store **sensitive data** (API keys, DB passwords)
  - Integrated với ECS Task Definition
  - Retrieve using `secrets` field in task definition
  - Encrypted at rest
  - Rotation support

**ECS Environment Variables:**

| Source | Use Case |
|--------|----------|
| Hardcoded | URLs, non-sensitive |
| SSM Parameter Store | Configs, some sensitive |
| **Secrets Manager** | Sensitive (passwords, keys) |
| S3 | Bulk data |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. S3 | Object storage, not secrets management |
| B. DynamoDB | Database, not secrets management |
| D. CloudWatch | Monitoring/logging, not secrets |

---

### Question 20:
Which feature allows CloudFront to deliver content from a private ALB in a VPC?

```
A. VPC Endpoints
B. VPC Origins
C. Origin Shield
D. Private Origins
```

### ✅ Answer: **B. VPC Origins**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **VPC Origins** (Private Origins):
  - Deliver content from **private VPC** resources
  - No need to expose on Internet
  - Supports:
    - Application Load Balancer
    - Network Load Balancer
    - EC2 Instances
  - Uses AWS PrivateLink

**Use case:**
- Internal applications
- Private APIs
- No public exposure needed

**Configuration:**
1. Create VPC origin in CloudFront
2. Set up PrivateLink/VPC endpoint
3. Route traffic through CloudFront

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. VPC Endpoints | Private access to AWS services, không phải CloudFront |
| C. Origin Shield | Reduces origin load, không phải VPC origins |
| D. Private Origins | Không phải tên chính thức |

---

## Summary Table

| Topic | Key Points |
|-------|------------|
| **Bastion Host** | Jump server in public subnet for SSH |
| **NACL** | Stateless, subnet-level, supports DENY |
| **Security Groups** | Stateful, instance-level, ALLOW only |
| **VPC Endpoint** | Gateway (free) for S3/DynamoDB |
| **NAT Gateway** | 5-100 Gbps, outbound Internet |
| **ECS Task** | Basic unit of work |
| **ECS IAM** | Instance Profile (agent), Task Role |
| **ECS Launch Types** | EC2 or Fargate |
| **Bind Mounts** | Container-to-container sharing |
| **ECR** | Backed by S3 |
| **CloudFront** | CDN, 600+ edge locations |
| **Origin Groups** | Failover between origins |
| **Invalidation** | Force cache refresh |
| **EKS** | Managed Kubernetes |
