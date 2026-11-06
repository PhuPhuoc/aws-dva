# Section 10 - AWS: Giám sát, xử lý sự cố và kiểm toán (Trang 478-537)

---

## Tổng quan về giám sát trên AWS (Trang 478-480)

- **CloudWatch**:
  - **Metrics**: thu thập và theo dõi các chỉ số quan trọng.
  - **Logs**: thu thập, giám sát, phân tích và lưu trữ tệp nhật ký.
  - **Events**: gửi thông báo khi có sự kiện nhất định trong AWS.
  - **Alarms**: phản ứng theo thời gian thực với metrics hoặc events.
- **X-Ray**: khắc phục sự cố hiệu năng và lỗi ứng dụng bằng cách theo dõi yêu cầu qua các microservices.
- **CloudTrail**: theo dõi các lệnh gọi API và kiểm toán thay đổi tài nguyên AWS do người dùng thực hiện.

## Amazon CloudWatch Metrics (Trang 481-485)

### Khái niệm và loại metric (Trang 481-482)

- CloudWatch cung cấp metrics cho các dịch vụ AWS.
- **Metric** là biến số cần theo dõi, ví dụ `CPUUtilization` hoặc `NetworkIn`.
- **Namespace** nhóm metrics, thường tương ứng với tên dịch vụ, ví dụ EC2 hoặc RDS.
- **Dimension** là thuộc tính của metric, ví dụ instance ID hoặc môi trường; mỗi metric hỗ trợ tối đa **30 dimensions**.
- Metrics có dấu thời gian; có thể tạo CloudWatch dashboards để hiển thị metrics.
- CloudWatch hỗ trợ hai loại metrics:
  - **Standard**: được EC2 và hầu hết dịch vụ khác hỗ trợ.
  - **Custom**: thường dùng cho EC2 và các instance tại chỗ.
- Với EC2, standard metrics theo dõi CPU và hiệu năng đĩa như IOPS, số byte đọc/ghi mỗi giây; không theo dõi mức sử dụng RAM hoặc phần trăm dung lượng đĩa đã dùng.
- Custom metrics có thể theo dõi RAM, dung lượng đĩa đã dùng, chỉ số ứng dụng như trạng thái chạy, số người dùng hoạt động, số lỗi và các giá trị tự định nghĩa.

> *Sơ đồ (Trang 483): Ví dụ dashboard CloudWatch hiển thị metrics.*

### Giám sát chi tiết EC2 (Trang 484)

- Mặc định, EC2 gửi standard metrics mỗi **5 phút**.
- Bật detailed monitoring để tăng tần suất gửi lên **1 phút**.
- Dùng detailed monitoring nếu cần ASG phản ứng và mở rộng nhanh hơn.
- AWS Free Tier cho phép dùng **10 detailed monitoring metrics**.

### Custom metrics (Trang 485)

- Có thể gửi custom metrics bằng API `PutMetricData` qua SDK, CLI hoặc gọi AWS API trực tiếp.
- **CloudWatch Agent** là cách được khuyến nghị để gửi metrics từ EC2.
- Có thể thêm dimensions như tên môi trường hoặc hệ thống.
- `StorageResolution` hỗ trợ:
  - **Standard**: 1 phút.
  - **High resolution**: 1, 5, 10 hoặc 30 giây; chi phí cao hơn.

## CloudWatch Logs (Trang 486-498)

### Khái niệm, lưu trữ và nguồn nhật ký (Trang 486-488)

- CloudWatch Logs thu thập, giám sát và lưu trữ tệp nhật ký từ tài nguyên AWS và hệ thống tại chỗ, giúp theo dõi ứng dụng, khắc phục sự cố và lưu nhật ký phục vụ kiểm toán/tuân thủ.
- **Log group** chứa các nhật ký liên quan đến một dịch vụ hoặc ứng dụng.
- **Log stream** là chuỗi các sự kiện nhật ký riêng lẻ bên trong log group.
- Có thể đặt chính sách hết hạn: không bao giờ hết hạn hoặc từ **1 ngày đến 10 năm**.
- Có thể gửi logs đến S3 để xuất, Kinesis Data Streams, Kinesis Data Firehose, Lambda hoặc OpenSearch.
- Logs được mã hóa mặc định; có thể cấu hình mã hóa bằng KMS key riêng.
- Nguồn logs gồm SDK, CloudWatch Logs Agent, CloudWatch Unified Agent, Elastic Beanstalk, ECS, Lambda, VPC Flow Logs, API Gateway, CloudTrail và truy vấn DNS của Route 53.

### CloudWatch Logs Insights (Trang 489-490)

> *Sơ đồ (Trang 489): Giao diện CloudWatch Logs Insights để truy vấn dữ liệu nhật ký.*

- Tìm kiếm và phân tích dữ liệu trong CloudWatch Logs bằng ngôn ngữ truy vấn chuyên dụng.
- Tự nhận diện các trường từ dịch vụ AWS và sự kiện nhật ký JSON.
- Có thể chọn trường, lọc điều kiện, tính thống kê tổng hợp, sắp xếp và giới hạn kết quả.
- Có thể lưu truy vấn, thêm truy vấn vào CloudWatch dashboards và truy vấn nhiều log groups ở các tài khoản AWS khác nhau.
- Đây là công cụ truy vấn, không phải công cụ xử lý theo thời gian thực.

### Xuất logs và đăng ký nhận logs (Trang 491-494)

- Khi xuất logs sang S3, dữ liệu có thể mất đến **12 giờ** mới sẵn sàng; API dùng là `CreateExportTask`.
- Xuất sang S3 không phù hợp cho xử lý gần thời gian thực. Dùng **Logs Subscriptions** để chuyển log events gần thời gian thực đến Kinesis Data Streams, Kinesis Data Firehose hoặc Lambda.
- **Subscription Filter** chọn những log events sẽ chuyển đến đích.

> *Sơ đồ (Trang 493): Tổng hợp CloudWatch Logs giữa nhiều tài khoản và Region.*

- **Cross-Account Subscription** gửi log events đến tài nguyên ở tài khoản AWS khác, chẳng hạn Kinesis Data Streams hoặc Kinesis Data Firehose.

### CloudWatch Logs trên EC2 và CloudWatch Agent (Trang 495-497)

- Mặc định, logs trên máy EC2 không được gửi đến CloudWatch.
- Cài CloudWatch Agent trên EC2 để đẩy các tệp logs cần thu thập; cần có IAM permissions phù hợp.
- Agent cũng có thể chạy trên máy chủ tại chỗ.
- CloudWatch Unified Agent thu thập custom metrics và logs từ EC2 hoặc máy chủ tại chỗ. Cần:
  - Cài đặt và cấu hình agent đúng cách.
  - IAM Role hoặc access key với quyền tối thiểu theo policy `CloudWatchAgentServerPolicy`.
  - Kết nối đến CloudWatch qua Internet hoặc VPC Endpoints.
  - Tệp cấu hình chỉ rõ metrics và logs cần thu thập.
- Agent có thể thu thập:
  - CPU: active, guest, idle, system, user và steal.
  - Đĩa: dung lượng free/used/total; Disk I/O gồm writes/reads, bytes và IOPS.
  - RAM: free, inactive, used, total và cached.
  - Netstat: số kết nối TCP/UDP, gói mạng và bytes.
  - Processes: total, dead, blocked, idle, running và sleep.
  - Swap Space: free, used và tỷ lệ sử dụng.
- Metrics EC2 có sẵn chỉ cung cấp thông tin tổng quát về đĩa, CPU và mạng.

### CloudWatch Logs Metric Filter (Trang 498)

- Có thể dùng filter expressions để tìm nội dung trong logs, ví dụ IP cụ thể hoặc số lần xuất hiện của `ERROR`.
- Metric filters có thể tạo metrics để kích hoạt CloudWatch Alarms.
- Filter không áp dụng ngược lên dữ liệu cũ; chỉ tạo metric datapoints cho các sự kiện phát sinh sau khi filter được tạo.
- Có thể chỉ định tối đa **3 dimensions** cho metric filter.

## CloudWatch Alarms và Synthetics (Trang 499-504)

### CloudWatch Alarms (Trang 499-503)

- Alarms có thể theo dõi metrics và kích hoạt hành động.
- Các trạng thái của alarm: `OK`, `INSUFFICIENT_DATA`, `ALARM`.
- **Period** là khoảng thời gian tính bằng giây dùng để đánh giá metric. Với custom metrics độ phân giải cao, period có thể là 10 giây, 30 giây hoặc bội số của 60 giây.
- Hành động của alarm có thể:
  - Dừng, kết thúc, khởi động lại hoặc khôi phục EC2 instance.
  - Kích hoạt hành động Auto Scaling.
  - Gửi thông báo đến SNS.
- **Composite Alarms** theo dõi trạng thái của nhiều alarms bằng điều kiện AND/OR, giúp giảm nhiễu từ các alarms riêng lẻ.
- Có thể tạo alarms dựa trên metrics từ CloudWatch Logs Metric Filters.
- Để kiểm tra alarm và thông báo, có thể dùng CLI đặt alarm vào trạng thái `ALARM`:

```bash
aws cloudwatch set-alarm-state --alarm-name "myalarm" --state-value ALARM --state-reason "testing purposes"
```

### EC2 Instance Recovery (Trang 502)

- CloudWatch alarm có thể có hành động khôi phục EC2.
- **Instance status check** kiểm tra máy ảo EC2; **System status check** kiểm tra phần cứng bên dưới.
- Sau khi khôi phục, instance giữ nguyên private IP, public IP, Elastic IP, metadata và placement group.

### CloudWatch Synthetics Canary (Trang 504)

- Canary là script có thể cấu hình để giám sát API, URL và website bằng cách mô phỏng thao tác của khách hàng.
- Kiểm tra tính khả dụng và độ trễ của endpoint; có thể lưu thời gian tải và ảnh chụp giao diện.
- Tích hợp với CloudWatch Alarms; script viết bằng Node.js hoặc Python, chạy trình duyệt Google Chrome không giao diện.
- Có thể chạy một lần hoặc theo lịch định kỳ.

## Amazon EventBridge (Trang 505-509)

### Rules, event bus và lưu trữ sự kiện (Trang 505-507)

- EventBridge, trước đây là CloudWatch Events, hỗ trợ:
  - **Schedule**: chạy tác vụ theo lịch bằng biểu thức cron.
  - **Event Pattern**: tạo rule phản ứng khi dịch vụ phát sinh sự kiện.
- Rule có thể kích hoạt Lambda hoặc gửi thông điệp đến SQS/SNS.

> *Sơ đồ (Trang 506): Rule EventBridge khớp sự kiện và chuyển sự kiện đến đích được cấu hình.*

- Event bus có thể được tài khoản AWS khác truy cập bằng resource-based policies.
- Có thể lưu trữ tất cả hoặc một phần sự kiện gửi đến event bus, lưu vô thời hạn hoặc trong khoảng thời gian đặt trước, rồi phát lại sự kiện đã lưu.

### Schema Registry và Resource-based Policy (Trang 508-509)

- EventBridge có thể phân tích sự kiện trong event bus để suy ra schema.
- Schema Registry giúp tạo mã ứng dụng phù hợp với cấu trúc sự kiện đã biết; schema có thể được quản lý phiên bản.
- Resource-based policy quản lý quyền trên một event bus, ví dụ cho phép hoặc từ chối sự kiện từ tài khoản AWS hoặc Region khác.
- Một trường hợp sử dụng là tập trung sự kiện từ AWS Organization về một tài khoản hoặc Region.

## AWS X-Ray (Trang 510-528)

### Theo dõi và phân tích yêu cầu (Trang 510-514)

- X-Ray theo dõi và phân tích các yêu cầu đi qua ứng dụng, hiển thị luồng qua các dịch vụ để tìm điểm nghẽn, lỗi và vấn đề hiệu năng trong ứng dụng phân tán.
- Có thể dùng để phân tích phụ thuộc giữa microservices, hành vi yêu cầu, ngoại lệ, độ trễ so với SLA, throttling và người dùng bị ảnh hưởng.
- Tương thích với Lambda, Elastic Beanstalk, ECS, ELB, API Gateway, EC2 và máy chủ ứng dụng tại chỗ.
- **Tracing** theo dõi yêu cầu từ đầu đến cuối; mỗi thành phần xử lý yêu cầu bổ sung trace của mình. Trace gồm các segments và subsegments.
- Có thể thêm annotations để bổ sung thông tin và chọn ghi lại mọi yêu cầu hoặc chỉ một tỷ lệ/lưu lượng nhất định.
- Dùng IAM để cấp quyền và KMS để mã hóa dữ liệu khi lưu trữ.

> *Sơ đồ (Trang 511): Bản đồ dịch vụ X-Ray thể hiện đường đi của yêu cầu và các dịch vụ liên quan.*

### Bật X-Ray và xử lý sự cố (Trang 515-518)

- Nhập X-Ray SDK vào ứng dụng viết bằng Java, Python, Go, Node.js hoặc .NET; chỉ cần ít thay đổi mã.
- SDK có thể thu thập lệnh gọi dịch vụ AWS, yêu cầu HTTP/HTTPS, lệnh gọi cơ sở dữ liệu như MySQL, PostgreSQL, DynamoDB và lệnh gọi hàng đợi như SQS.
- Cài X-Ray daemon hoặc bật tích hợp X-Ray có sẵn của dịch vụ AWS. Daemon nhận dữ liệu trace qua UDP; ứng dụng cần IAM permissions để ghi dữ liệu vào X-Ray.
- X-Ray tổng hợp dữ liệu từ các dịch vụ để tạo service map từ segments và traces.
- Nếu X-Ray không hoạt động trên EC2, kiểm tra IAM Role và X-Ray daemon.
- Để bật trên Lambda, cần execution role có `AWSXRayWriteOnlyAccess`, import X-Ray trong mã và bật Active Tracing.
- **Instrumentation** là việc đo hiệu năng, chẩn đoán lỗi và ghi thông tin trace. X-Ray SDK hỗ trợ cấu hình hoặc tùy chỉnh dữ liệu bằng interceptors, filters, handlers và middleware.

### Các khái niệm và sampling (Trang 519-521)

| Khái niệm | Ý nghĩa |
|-----------|---------|
| Segment | Dữ liệu một ứng dụng/dịch vụ gửi về X-Ray |
| Subsegment | Chi tiết bổ sung bên trong một segment |
| Trace | Tập hợp các segments của một yêu cầu từ đầu đến cuối |
| Sampling | Giảm lượng yêu cầu gửi đến X-Ray để giảm dữ liệu và chi phí |
| Annotation | Cặp khóa-giá trị được lập chỉ mục, có thể lọc khi tìm traces |
| Metadata | Cặp khóa-giá trị không được lập chỉ mục để tìm kiếm |

- Daemon có thể được cấu hình gửi traces giữa các tài khoản; cần IAM permissions để daemon assume role.
- Sampling rules kiểm soát lượng dữ liệu được ghi và có thể thay đổi mà không sửa mã.
- Mặc định, X-Ray SDK ghi yêu cầu đầu tiên mỗi giây, sau đó lấy mẫu **5%** các yêu cầu bổ sung. Một yêu cầu mỗi giây là reservoir.
- Có thể tạo sampling rules riêng với reservoir và rate.

### X-Ray APIs và tích hợp (Trang 522-528)

- Các write APIs dùng bởi X-Ray daemon:
  - `PutTraceSegments`: tải segment documents lên X-Ray.
  - `PutTelemetryRecords`: tải telemetry của daemon, như số segment nhận/từ chối và lỗi kết nối backend.
  - `GetSamplingRules`, `GetSamplingTargets`, `GetSamplingStatisticSummaries`: lấy sampling rules và dữ liệu sampling.
- Các read APIs:
  - `GetServiceGraph`: lấy service graph chính.
  - `BatchGetTraces`: lấy danh sách traces theo ID.
  - `GetTraceSummaries`: lấy IDs và annotations của traces trong khoảng thời gian, có thể kèm filter.
  - `GetTraceGraph`: lấy service graph cho các traces cụ thể.
- Elastic Beanstalk có X-Ray daemon; có thể bật qua console hoặc tệp `.ebextensions/xray-daemon.config`. Cần IAM permissions cho instance profile và instrument mã ứng dụng. Daemon không được cung cấp cho Multicontainer Docker.

> *Sơ đồ (Trang 525-526): Các lựa chọn tích hợp X-Ray với ECS và ví dụ task definition.*

- **AWS Distro for OpenTelemetry** là bản phân phối OpenTelemetry được AWS hỗ trợ, gồm APIs, thư viện, agents và collector.
- Distro thu thập distributed traces, metrics và metadata tài nguyên AWS; agents có thể tự instrument ứng dụng.
- Có thể gửi traces/metrics đến nhiều dịch vụ AWS hoặc đối tác như X-Ray, CloudWatch và Prometheus; hỗ trợ ứng dụng trên EC2, ECS, EKS, Fargate, Lambda và hệ thống tại chỗ.
- Có thể chuyển từ X-Ray khi cần chuẩn hóa theo OpenTelemetry hoặc gửi dữ liệu đồng thời đến nhiều đích.

> *Sơ đồ (Trang 528): Minh họa AWS Distro for OpenTelemetry thu thập và chuyển telemetry đến nhiều đích.*

## AWS CloudTrail (Trang 529-536)

### CloudTrail và các loại event (Trang 529-531)

- CloudTrail hỗ trợ quản trị, tuân thủ và kiểm toán tài khoản AWS; được bật mặc định.
- Lưu lịch sử events/lệnh gọi API trong tài khoản AWS từ Console, SDK, CLI và các dịch vụ AWS.
- Có thể đưa logs của CloudTrail vào CloudWatch Logs hoặc S3.
- Trail áp dụng cho tất cả Regions theo mặc định hoặc cho một Region cụ thể. Khi tài nguyên bị xóa, kiểm tra CloudTrail để tìm hoạt động liên quan.
- **Management Events** ghi lại thao tác quản lý tài nguyên, ví dụ cấu hình IAM, tạo subnet hoặc tạo trail. Trail ghi management events mặc định; có thể tách Read Events và Write Events.
- **Data Events** không được ghi mặc định do khối lượng lớn. Bao gồm hoạt động cấp đối tượng S3 như `GetObject`, `DeleteObject`, `PutObject` và hoạt động thực thi Lambda qua API `Invoke`.
- **CloudTrail Insights Events** phát hiện hoạt động bất thường trong tài khoản.

> *Sơ đồ (Trang 530): Luồng CloudTrail ghi nhận hoạt động tài khoản AWS.*

### CloudTrail Insights và thời gian lưu (Trang 532-533)

- Insights phát hiện dấu hiệu bất thường như cấp phát tài nguyên không phù hợp, chạm giới hạn dịch vụ, tăng đột biến các thao tác IAM hoặc gián đoạn hoạt động bảo trì định kỳ.
- CloudTrail Insights tạo baseline từ management events bình thường, sau đó liên tục phân tích write events để phát hiện mẫu bất thường.
- Anomaly xuất hiện trong CloudTrail console, được gửi đến S3 và tạo EventBridge event để tự động hóa.
- CloudTrail lưu events trong **90 ngày**. Để giữ lâu hơn, ghi logs vào S3 và dùng Athena để truy vấn.

> *Sơ đồ (Trang 534-535): EventBridge nhận diện lệnh gọi API và phối hợp với CloudTrail.*

### Phân biệt CloudTrail, CloudWatch và X-Ray (Trang 536)

| Dịch vụ | Mục đích chính |
|---------|----------------|
| CloudTrail | Kiểm toán lệnh gọi API của người dùng, dịch vụ và AWS Console; tìm nguyên nhân thay đổi hoặc lệnh gọi trái phép |
| CloudWatch | Theo dõi metrics theo thời gian, lưu application logs và gửi thông báo khi metrics bất thường |
| X-Ray | Phân tích traces và service map; tìm độ trễ, lỗi và điểm nghẽn trên yêu cầu phân tán |

## Bài tập trên slide (Trang 537)

- Theo dõi CPU Utilization của EC2 bằng CloudWatch Alarms.
- Tạo EventBridge Rule để nhận thông báo khi trạng thái EC2 instance thay đổi.
