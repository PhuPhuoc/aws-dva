# Section 7 - AWS Integration & Messaging (SQS, SNS, Kinesis), Monitoring (CloudWatch, X-Ray, CloudTrail), AWS Lambda

---

## Phần 1: Amazon SQS - Simple Queue Service (Trang 441-457)

### SQS - Consuming Messages (Trang 441-442)

**Quy trình Consumer (workflow):**
1. Poll (lấy) messages từ SQS (nhận nhiều nhất **10 messages** một lần)
2. Xử lý (process) messages (ví dụ: insert vào RDS)
3. Xóa messages bằng **DeleteMessage API**

**Xử lý song song (Parallel Processing):**
- Nhiều EC2 instances có thể consume (đọc) song song
- **Ghi ít nhất một lần (At least once delivery)**
- **Giữ thứ tự message tốt nhất có thể (Best-effort message ordering)**
- Scale ngang (horizontally) để cải thiện throughput

### Amazon SQS - Security (Trang 445-446)

**Mã hóa - Encryption:**
- **In-flight** (trên đường truyền): HTTPS API
- **At-rest** (khi lưu): KMS keys
- **Client-side**: Customer (khách hàng) tự quản lý

**Access Controls - kiểm soát truy cập:**
- **IAM policies**: Điều chỉnh SQS API access
- **SQS Access Policies**: Truy cập cross-account, cho phép SNS/S3 ghi vào queue

### SQS - Message Visibility Timeout (Trang 447-448)

- Sau khi được poll, message trở nên **invisible** với các consumer khác
- Default: **30 seconds**
- Message có 30 giây để process
- Nếu không delete, message **xuất hiện lại** trong queue
- Có thể dùng **ChangeMessageVisibility API** để thêm thời gian

**Lưu ý:**
- Timeout quá ngắn → duplicates
- Timeout quá dài → xử lý lại chậm khi hệ thống crash

### Amazon SQS - Dead Letter Queue (DLQ) (Trang 449-450)

- Nếu message fail (vượt qua visibility timeout), sẽ quay lại queue
- Ngưỡng **MaximumReceives** (số lần receive tối đa)
- Sau khi vượt ngưỡng → message được chuyển đến **DLQ**
- DLQ phải cùng loại (match) với queue gốc:
  - FIFO queue → FIFO DLQ
  - Standard queue → Standard DLQ
- Cấu hình thời gian lưu (retention) lên đến **14 ngày**

### SQS DLQ - Redrive to Source (Trang 450)

- Tính năng giúp đưa messages từ DLQ về lại source queue để xử lý
- Xác định nguyên nhân vấn đề
- Sửa (fix) code
- **Redrive** (đưa lại) messages về source queue

### Amazon SQS - Delay Queue (Trang 451)

- Trì hoãn (delay) message (không hiển thị) lên đến **15 phút**
- Mặc định: 0 giây (message có thể nhận ngay)
- Cấu hình ở cấp queue hoặc cho từng message bằng tham số **DelaySeconds**

### Amazon SQS - Long Polling (Trang 452)

- Consumer có thể **wait** (chờ) khi queue chưa có messages
- Giảm số lần gọi API, tăng hiệu quả
- Thời gian chờ (wait time): **1-20 giây** (nên dùng 20 giây)
- Bật ở cấp queue hoặc dùng tham số **WaitTimeSeconds**

### SQS Extended Client (Trang 453)

- Giới hạn kích thước message (message size limit): **256KB**
- Với messages lớn hơn (ví dụ: 1GB): Dùng SQS Extended Client (thư viện Java) để lưu message trong S3

### SQS Must Know API (Trang 454)

| API | Mô tả |
|-----|-------------|
| `CreateQueue` | Tạo queue (MessageRetentionPeriod) |
| `DeleteQueue` | Xóa queue |
| `PurgeQueue` | Xóa tất cả messages |
| `SendMessage` | Gửi (DelaySeconds) |
| `ReceiveMessage` | Nhận messages (max 10) |
| `DeleteMessage` | Xóa sau khi process |
| `ChangeMessageVisibility` | Thay đổi timeout |

### Amazon SQS - FIFO Queue (Trang 455-457)

**FIFO = First In First Out**

**Thuộc tính:**
- Throughput hạn chế: **300 msg/s** (không dùng batching)
- **3000 msg/s** (khi dùng batching)
- Khả năng gửi **chỉ một lần duy nhất (exactly-once)**
- Messages được xử lý **đúng thứ tự**

**Deduplication - loại bỏ trùng lặp:**
- Khoảng thời gian (interval): **5 phút**
- Methods:
  - **Content-based**: hash SHA-256 của nội dung (body) message
  - **Explicit**: Tự cung cấp Message Deduplication ID

**Message Grouping - nhóm messages:**
- Same **MessageGroupID** → một consumer, ordered
- Khác MessageGroupID → xử lý song song giữa các group, thứ tự được giữ trong từng group

---

## Phần 2: Amazon SNS - Simple Notification Service (Trang 458-468)

### Amazon SNS Overview (Trang 459)

**Kiến trúc (Architecture):**
- **Producer (người gửi)** gửi đến một SNS topic
- Nhiều **subscribers (người nhận)** nhận tất cả messages
- Nhiều nhất **12,500,000 subscriptions** mỗi topic
- Giới hạn **100,000 topics** (mỗi tài khoản)

**Các loại subscriber:**
- SQS, Lambda, Kinesis Data Firehose (các service AWS)
- Email, SMS, HTTP(S)
- Mobile Notifications

### SNS Integrations (Trang 460)

**AWS Services có thể gửi đến SNS:**
- CloudWatch Alarms
- AWS Budgets
- Auto Scaling Group
- S3 Bucket Events
- CloudFormation State Changes
- AWS DMS
- Lambda, DynamoDB, RDS Events

### Amazon SNS - How to Publish (Trang 461)

**Publish đến Topic:**
1. Tạo một topic
2. Tạo subscriptions
3. Publish đến topic

**Direct Publish (cho mobile apps):**
1. Tạo application nền tảng (platform application)
2. Tạo endpoint nền tảng (platform endpoint)
3. Publish đến platform endpoint

### Amazon SNS - Security (Trang 462)

**Mã hóa:**
- In-flight (trên đường truyền): HTTPS
- At-rest (khi lưu): KMS keys
- Client-side: Customer (khách hàng) tự quản lý

**Kiểm soát truy cập (Access Controls):**
- IAM policies
- SNS Access Policies (cross-account - giữa các tài khoản)

### SNS + SQS: Fan Out (Trang 463)

**Pattern - mô hình:**
```
SNS Topic → SQS Queue 1 → Consumer 1
         → SQS Queue 2 → Consumer 2
         → SQS Queue 3 → Consumer 3
```

**Lợi ích:**
- Push một lần, nhận ở tất cả queues
- Tách biệt hoàn toàn (fully decoupled), không mất dữ liệu (no data loss)
- SQS cung cấp lưu trữ bền vững (persistence) và cơ chế thử lại (retries)
- Hỗ trợ chuyển message đến nhiều Region (cross-Region)

### Amazon SNS - FIFO Topic (Trang 466)

- Thứ tự xử lý FIFO (đầu vào trước, đầu ra trước)
- Giống SQS FIFO:
  - Dùng Message Group ID để giữ thứ tự
  - Deduplication (loại bỏ trùng lặp)
- Có thể dùng cả SQS Standard lẫn FIFO queue làm subscriber

### SNS - Message Filtering (Trang 468)

- Dùng JSON policy (filter policy) để lọc messages
- Subscription không có filter → nhận tất cả messages

---

## Phần 3: Amazon Kinesis (Trang 469-475)

### Kinesis Data Streams (Trang 469-470)

**Tính năng:**
- Thu thập và lưu trữ (collect and store) dữ liệu streaming **real-time**
- Thời gian lưu (retention): **1-365 ngày**
- Ability to **replay** data - khả năng phát lại dữ liệu
- Dữ liệu lên đến **1MB** (data up to 1MB)
- Đảm bảo thứ tự (ordering) với cùng Partition ID
- Mã hóa KMS khi lưu trữ (at-rest)

### Kinesis Data Streams - Capacity Modes (Trang 471)

**Chế độ Provisioned (Provisioned Mode):**
- Tự chọn số **shards**
- Mỗi shard: **1MB/s vào**, **2MB/s ra**
- Tự scale (scale thủ công)
- Thanh toán theo shard theo giờ

**Chế độ On-Demand (On-Demand Mode):**
- Không cần provision (chuẩn bị sẵn capacity)
- Capacity mặc định: **4 MB/s**
- Tự động scale theo lượng đỉnh (peak)
- Thanh toán theo stream + dữ liệu vào/ra

### Amazon Data Firehose (Trang 472-473)

- Đưa (load) dữ liệu streaming đến các destinations
- Destinations:
  - S3, Redshift, OpenSearch
  - 3rd party (Splunk, MongoDB, Datadog)
  - Custom HTTP
- **Fully managed** (quản lý hoàn toàn), serverless
- Gần như real-time (near real-time)
- Hỗ trợ định dạng CSV, JSON, Parquet, Avro
- Biến đổi dữ liệu tùy chỉnh (custom transformations) bằng Lambda

### Kinesis Data Streams vs Firehose (Trang 474)

| Đặc điểm | Kinesis Data Streams | Firehose |
|---------|---------------------|----------|
| **Loại (Type)** | Thu thập streaming (Streaming collection) | Đưa dữ liệu đến destinations (Load to destinations) |
| **Code** | Producer & Consumer (phải tự viết) | Fully managed (quản lý hoàn toàn) |
| **Real-time** | Có | Near real-time |
| **Capacity** | Provisioned hoặc On-demand | Auto scaling (tự động) |
| **Replay** | Có | Không |
| **Storage (Lưu trữ)** | Lâu nhất 365 ngày | Không |

---

## Phần 4: SQS vs SNS vs Kinesis (Trang 476)

| Đặc điểm | SQS | SNS | Kinesis |
|---------|-----|-----|---------|
| **Loại (Type)** | Hàng đợi (Queue) | Pub/Sub | Dòng (Stream) |
| **Giao nhận (Delivery)** | Kéo (Pull) | Đẩy (Push) | Kéo (Pull) |
| **Consumers** | 1 | Nhiều | Nhiều (theo shards) |
| **Thứ tự (Ordering)** | Có FIFO | Có FIFO | Theo từng shard |
| **Replay** | Không | Không | Có |
| **Dùng cho (Use case)** | Tách biệt (Decouple) | Phân tán (Fan-out) | Phân tích real-time |

---

## Phần 5: Amazon CloudWatch (Trang 481-504)

### Amazon CloudWatch Metrics (Trang 481-482)

**Khái niệm:**
- **Namespace**: nhóm (container) cho metrics (ví dụ: AWS/EC2)
- **Dimension**: thuộc tính (attribute) của metric (tối đa 30)
- **Metric**: chỉ số (biến số) để giám sát (monitor) (CPUUtilization, NetworkIn)
- **Timestamp**: thời điểm đo lường (thời gian)

**EC2 Metrics:**
- Chuẩn: CPU, Disk (đĩa), Network (mạng)
- Theo mặc định (default) **không thể monitor RAM (memory)**
- Tự tạo custom metrics cho RAM, disk usage

### EC2 Detailed Monitoring (Trang 484)

- Standard: **5 minutes**
- Detailed: **1 minute**
- Free Tier: 10 metrics detailed monitoring (miễn phí)
- Dùng để Auto Scaling Group scale nhanh hơn

### CloudWatch Custom Metrics (Trang 485)

**Hai cách:**
- **PutMetricData API** (SDK, CLI)
- **CloudWatch Agent** (khuyến khích dùng cho EC2)

**Resolution - độ phân giải:**
- Standard: **1 phút**
- High Resolution (độ phân giải cao): **1/5/10/30 giây** (chi phí cao hơn)

### CloudWatch Logs (Trang 486-488)

**Khái niệm:**
- **Log groups**: nhóm (container) chứa các logs liên quan (related logs)
- **Log stream**: một chuỗi (sequence) logs riêng lẻ
- Thời hạn hết (expiration): không bao giờ đến 10 năm

**Sources - các nguồn:**
- SDK, CloudWatch Logs Agent, Unified Agent
- Elastic Beanstalk, ECS, Lambda
- VPC Flow Logs, API Gateway
- CloudTrail, Route53

### CloudWatch Logs Insights (Trang 489-490)

- Ngôn ngữ query chuyên dụng (purpose-built)
- Tự phát hiện (auto-discover) các trường (fields) từ JSON logs
- Tính thống kê (statistics), lọc, sắp xếp
- Lưu các query (save queries), thêm vào dashboards
- Query giữa nhiều accounts/regions

### CloudWatch Logs for EC2 (Trang 495-496)

**Yêu cầu:**
1. Chạy (run) CloudWatch agent trên EC2
2. IAM permissions đúng
3. File cấu hình cho metrics/logs

**Unified Agent thu thập:**
- CPU, metrics đĩa, RAM
- Netstat, Processes (tiến trình), Swap Space

### CloudWatch Logs Metric Filter (Trang 498)

- Biểu thức lọc (filter expressions) cho logs
- Kích hoạt (trigger) CloudWatch Alarms
- **Không lọc ngược (does not retroactively filter)** - không lọc dữ liệu cũ

### CloudWatch Alarms (Trang 499-503)

**States - các trạng thái:**
- **OK**
- **INSUFFICIENT_DATA**
- **ALARM**

**Targets - các mục tiêu:**
- Dừng, kết thúc, khởi động lại, khôi phục (Stop, Terminate, Reboot, Recover) EC2
- Kích hoạt (trigger) Auto Scaling
- Gửi (send) thông báo SNS

**Khôi phục EC2 Instance (EC2 Instance Recovery):**
- **Instance status**: Kiểm tra (check) phần mềm/VM của EC2
- **System status**: Kiểm tra (check) phần cứng
- Recovery giữ nguyên IP và metadata của instance

**Composite Alarms (alarm tổng hợp):**
- Giám sát (monitor) nhiều alarm
- Điều kiện AND/OR (và/or)
- Giảm nhiễu (alarm noise) từ các alarm

### CloudWatch Synthetics Canary (Trang 504)

- Scripts cấu hình được để monitor (giám sát) APIs, URLs, Websites
- Tái tạo (reproduce) hành vi của khách hàng (user)
- Kiểm tra khả dụng (availability) và độ trễ (latency)
- Tích hợp với CloudWatch Alarms
- Scripts viết bằng Node.js hoặc Python

---

## Phần 6: Amazon EventBridge (Trang 505-509)

### Amazon EventBridge Overview (Trang 505)

**Hai patterns - hai mô hình:**
- **Schedule**: Cron jobs (các scripts chạy theo lịch)
- **Event Pattern (Mẫu sự kiện)**: phản ứng (react) với các service events

**Targets - các mục tiêu:**
- Các Lambda functions (hàm)
- SQS/SNS messages

### Amazon EventBridge Features (Trang 507-509)

**Event Buses (các tổng sự kiện):**
- Event bus mặc định cho các AWS services
- Partner event buses (dành cho SaaS)
- Custom event buses (tự tạo)

**Lưu trữ & Phát lại (Archive & Replay):**
- Lưu trữ (archive) events (toàn bộ hoặc đã lọc)
- Phát lại (replay) các events đã archive

**Schema Registry (đăng ký schema dữ liệu):**
- Infer schema từ events
- Tạo (generate) code với cấu trúc đã biết trước
- Hỗ trợ quản lý phiên bản (versioning)

**Resource-based Policies (chính sách dựa trên resource):**
- Truy cập cross-account (giữa các tài khoản)
- Gom (aggregate) events từ Organization

---

## Phần 7: AWS X-Ray (Trang 510-528)

### AWS X-Ray Overview (Trang 510-512)

**X-Ray làm gì:**
- Theo dõi và phân tích (trace & analyze) requests
- Trực quan hóa (visualize) luồng request giữa các service
- Nhận diện (identify) điểm nghẽn (bottlenecks)
- Khắc phục (troubleshoot) các vấn đề hiệu suất (performance)

**Lợi ích:**
- Chẩn đoán (troubleshooting) hiệu năng
- Hiểu các phụ thuộc (dependencies) giữa microservices
- Định vị (pinpoint) các vấn đề của service
- Xem xét (review) hành vi của request
- Tìm lỗi (errors) và ngoại lệ (exceptions)
- Giám sát (monitoring) SLA

### X-Ray Compatibility (Trang 513)

- AWS Lambda, Elastic Beanstalk, ECS
- ELB, API Gateway
- EC2 Instances và hệ thống on-premises (nội bộ)

### AWS X-Ray - How it Works (Trang 514-516)

**Components - các thành phần:**
1. **SDK** cấy vào (cài đặt) trong code ứng dụng
2. **X-Ray Daemon** (thiết bị chặn và chuyển tiếp UDP packets - UDP packet interceptor)

**SDK captures - SDK bắt:**
- Các cuộc gọi (calls) tới AWS services
- Các request HTTP/HTTPS
- Các cuộc gọi database
- Các cuộc gọi queue

### X-Ray Concepts (Trang 519)

| Khái niệm | Mô tả |
|---------|-------------|
| **Segment** | Dữ liệu từ application/service |
| **Subsegment** | Chi tiết hơn (more details) trong segment |
| **Trace** | Tập hợp (collection) các segments |
| **Sampling** | Giảm dữ liệu được ghi (reduce data recorded) |
| **Annotations** | Các cặp key-value, được lập chỉ mục (indexed) |
| **Metadata** | Các cặp key-value, không lập chỉ mục (not indexed) |

**Sampling theo mặc định (default sampling):**
- Request đầu tiên của mỗi giây
- 5% số request còn lại

### AWS Distro for Open Telemetry (Trang 527-528)

- Phân phối (distribution) OpenTelemetry do AWS hỗ trợ
- Bộ APIs, thư viện (libraries), agents dùng chung duy nhất
- Thu thập (collect) traces và metrics
- Gửi đến nhiều destinations (X-Ray, CloudWatch, Prometheus)

---

## Phần 8: AWS CloudTrail (Trang 529-535)

### AWS CloudTrail Overview (Trang 529-530)

**CloudTrail làm gì:**
- Governance, compliance, audit - quản trị, tuân thủ, kiểm toán
- **Bật theo mặc định (enabled by default)**
- Lịch sử (history) các API calls

**Sources - các nguồn:**
- Console, SDK, CLI, các AWS Services

**Tích hợp (Integrations):**
- CloudWatch Logs
- S3

### CloudTrail Events (Trang 531-532)

**Management Events (sự kiện quản lý):**
- Các thao tác (operations) trên resources
- Cấu hình bảo mật (security), routing, logging
- Phân biệt sự kiện đọc (read) và ghi (write)

**Data Events (sự kiện dữ liệu):**
- Cấp object trong S3 (GetObject, DeleteObject, PutObject)
- Chạy (execution) Lambda function
- Không được ghi log theo mặc định (vì số lượng lớn)

**CloudTrail Insights:**
- Phát hiện (detect) các hoạt động bất thường (unusual activity)
- Cấu hình (provisioning) không chính xác
- Vượt giới hạn (hitting limits)
- Gặp sự bùng phát (bursts) các thao tác IAM

### CloudTrail Events Retention (Trang 533)

- Events được lưu **90 ngày**
- Ghi log vào S3 + Athena để lưu lâu hơn

### CloudTrail vs CloudWatch vs X-Ray (Trang 536)

| Service | Purpose - Mục đích |
|---------|---------|
| **CloudTrail** | Kiểm toán (audit) API calls, phát hiện truy cập không được cấp phép (unauthorized access) |
| **CloudWatch** | Metrics theo thời gian, logs, alarms |
| **X-Ray** | Distributed tracing (theo dõi phân tán), phân tích latency |

---

## Phần 9: AWS Lambda (Trang 539-590)

### What's Serverless? (Trang 540)

- Không cần quản lý (manage) bất kỳ server nào
- Chỉ cần triển khai (deploy) code/functions
- FaaS (Function as a Service - dịch vụ dạng hàm)
- Các databases, messaging, storage do AWS quản lý sẵn

### Why AWS Lambda (Trang 542)

**vs EC2:**

| EC2 | Lambda |
|-----|--------|
| Máy ảo (Virtual servers) | Hàm ảo (Virtual functions) |
| Giới hạn bởi RAM/CPU | Giới hạn bởi thời gian |
| Chạy liên tục | Chạy theo yêu cầu (on-demand) |
| Scale thủ công (manual) | Scale tự động (automated) |

### AWS Lambda Benefits (Trang 543)

- **Giá đơn giản (Easy Pricing)**: Trả tiền theo số request và thời gian tính toán
- **Free Tier**: 1,000,000 requests + 400,000 GB-seconds
- Integrated với AWS services
- Hỗ trợ nhiều ngôn ngữ lập trình
- Giám sát (monitoring) dễ dàng (CloudWatch)
- Up to **10GB RAM**

### AWS Lambda Language Support (Trang 544)

- Node.js, Python, Java
- C#, PowerShell, Ruby
- Custom Runtime API (tự định nghĩa runtime)
- Container Image (qua Lambda Runtime API)

### Lambda Pricing (Trang 548)

**Trả tiền theo số lần gọi (Pay per calls):**
- 1 triệu request đầu: **miễn phí**
- Vượt mức: $0.20 cho mỗi 1 triệu request ($0.0000002/request)

**Trả tiền theo thời gian thực thi (Pay per duration):**
- 400,000 GB-seconds miễn phí mỗi tháng
- Vượt mức: $1.00 cho mỗi 600,000 GB-seconds

### Lambda - Synchronous Invocations (Trang 549-550)

**Services - các dịch vụ:**
- CLI, SDK, API Gateway, ALB
- CloudFront (Lambda@Edge)
- S3 Batch
- Cognito, Step Functions
- Lex, Alexa, Kinesis Data Firehose

**Xử lý lỗi:** Retries (thử lại) ở phía client

### Lambda - Asynchronous Invocations (Trang 556)

**Services:**
- S3, SNS, CloudWatch Events/EventBridge
- CodeCommit, CodePipeline
- CloudWatch Logs, SES, CloudFormation

**Retry behavior:**
- Thử tối đa 3 lần
- Chờ 1 phút, sau đó chờ 2 phút trước mỗi lần thử
- Cần xử lý **idempotent** (chạy lại cho kết quả như nhau)
- DLQ (Dead Letter Queue) cho các lần xử lý thất bại

### Lambda - Event Source Mapping (Trang 561)

**Sources - các nguồn:**
- Dữ liệu từ Kinesis Data Streams
- SQS và SQS FIFO
- DynamoDB Streams

**Lambda được gọi một cách đồng bộ (synchronously)**

### Lambda - Destinations (Trang 565)

**Async invocations:**
- Xử lý thành công: gửi đến SQS, SNS, Lambda, EventBridge
- Xử lý thất bại: chuyển đến DLQ

**Event source mapping (lớp ánh xạ nguồn sự kiện):**
- Batch bị bỏ (discarded): gửi thông báo đến SQS, SNS

### Lambda Execution Role (Trang 566)

**Managed policies (các policy đã quản lý sẵn):**
- `AWSLambdaBasicExecutionRole` - CloudWatch Logs
- `AWSLambdaKinesisExecutionRole` - Kinesis
- `AWSLambdaDynamoDBExecutionRole` - DynamoDB Streams
- `AWSLambdaSQSQueueExecutionRole` - SQS
- `AWSLambdaVPCAccessExecutionRole` - VPC
- `AWSXRayDaemonWriteAccess` - X-Ray

### Lambda Resource Based Policies (Trang 567)

- Cấp quyền cho accounts/services khác được sử dụng Lambda
- Tương tự như S3 bucket policies

### Lambda Logging & Monitoring (Trang 569)

**CloudWatch Logs:**
- Log thực thi (execution logs) được lưu tại đây
- IAM policy cho phép ghi (writes) logs

**CloudWatch Metrics:**
- Số lần chạy (invocations), thời gian thực thi (durations)
- Số lần thực thi đồng thời (concurrent executions)
- Số lỗi, tỷ lệ thành công
- Số lần bị throttle, Iterator Age (tuổi của iterator)

### Lambda Tracing with X-Ray (Trang 570)

- Bật (enable) Active Tracing
- IAM: `AWSXRayDaemonWriteAccess`
- Các biến môi trường (environment variables) cho X-Ray

### CloudFront Functions & Lambda@Edge (Trang 571-576)

**CloudFront Functions:**
- Chỉ hỗ trợ JavaScript (nhanh và nhẹ hơn)
- Khởi động dưới 1ms (sub-ms), hàng triệu req/s
- Các giai đoạn Viewer Request/Response
- Chuẩn hóa (normalize) cache key, chỉnh sửa (manipulation) HTTP header

**Lambda@Edge:**
- Hỗ trợ Node.js hoặc Python
- Hàng nghìn req/s
- 4 giai đoạn: Viewer/Origin Request/Response
- Viết (author) ở us-east-1, sau đó được nhân rộng (replicate) toàn cầu

### Lambda in VPC (Trang 577-579)

**Theo mặc định:** Lambda chạy (runs) ở bên ngoài VPC của bạn

**Để truy cập VPC resources:**
1. Định nghĩa (define) VPC ID, Subnets, Security Groups
2. Lambda tự tạo (create) ENI trong các subnets
3. Bắt buộc cần (requires) `AWSLambdaVPCAccessExecutionRole`

**Internet Access - truy cập Internet:**
- Public subnet: KHÔNG có internet access
- Private subnet: ra Internet qua NAT Gateway
- **VPC Endpoints**: truy cập riêng tư (private) tới các AWS services

### Lambda Function Configuration (Trang 580)

**RAM:**
- 128MB đến **10GB** (tăng theo bước (increment) 1MB)
- Càng nhiều RAM = càng nhiều vCPU
- Ở mức 1792MB tương đương 1 vCPU đầy đủ

**Timeout:** Mặc định 3 giây, tối đa **900 giây (15 phút)**

### Lambda Execution Context (Trang 581)

- Môi trường runtime tạm thời (temporary)
- Khởi tạo (initialize) các dependencies bên ngoài
- **Được tái sử dụng cho các lần invoke tiếp theo**
- Chứa sẵn thư mục `/tmp`

### Lambda /tmp Space (Trang 583)

- Dung lượng lưu trữ lên đến **10GB**
- Nội dung vẫn còn khi instance bị đóng băng (frozen)
- Dùng cho cache tạm (transient cache)
- Để lưu trữ lâu dài (permanent): dùng S3
- Mã hóa (encrypt) bằng KMS Data Keys

### Lambda Layers (Trang 584)

- Đưa dependencies ra bên ngoài (externalize) khỏi deployment package
- Chạy custom runtimes (C++, Rust)
- Tái sử dụng (reuse) giữa nhiều functions

### Lambda File Systems (Trang 585)

- Truy cập (access) EFS nằm trong VPC
- Cấu hình EFS Access Points (điểm truy cập)
- Lưu ý (watch) giới hạn số kết nối EFS

### Lambda Concurrency (Trang 587-588)

- Mức mặc định: **1000 executions đồng thời (concurrent)**
- **Reserved concurrency**: dành trước (limit) mức độ song song cho từng function
- Hành vi khi bị throttle (Throttle behavior):
  - Đồng bộ (Sync): trả về ThrottleError (429)
  - Bất đồng bộ (Async): Retry (thử lại) → DLQ

### Cold Starts & Provisioned Concurrency (Trang 590)

**Cold Start:**
- Instance mới: tải code (load), chạy init (khởi tạo)
- Request đầu tiên có độ trễ (latency) cao hơn

**Provisioned Concurrency:**
- Instance được cấp phát (allocated) trước khi có invocation
- Không còn cold starts
- Latency thấp liên tục (mọi lúc)
- Hỗ trợ Application Auto Scaling (tự động điều chỉnh số lượng)
