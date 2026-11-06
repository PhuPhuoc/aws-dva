# Section 11 - AWS Lambda (Trang 538-590)

---

## Tổng quan về serverless và Lambda (Trang 538-544)

### Serverless và các dịch vụ serverless của AWS (Trang 538-541)

- **Serverless** là mô hình trong đó developer triển khai mã hoặc functions mà không phải tự quản lý hay cấp phát máy chủ.
- Serverless ban đầu gắn với **FaaS (Function as a Service)**; hiện còn bao gồm các dịch vụ được AWS quản lý như databases, messaging và storage. Serverless không có nghĩa là không có máy chủ, mà là người dùng không phải quản lý chúng.
- Các dịch vụ serverless được slide liệt kê: Lambda, DynamoDB, Cognito, API Gateway, S3, SNS, SQS, Kinesis Data Firehose, Aurora Serverless, Step Functions và Fargate.

> *Sơ đồ (Trang 541): Các dịch vụ serverless trong AWS.*

### Lambda so với EC2 và lợi ích (Trang 542-543)

| Đặc điểm | EC2 | Lambda |
|---------|-----|--------|
| Mô hình | Máy chủ ảo | Function ảo, không phải tự quản lý máy chủ |
| Giới hạn chính | RAM và CPU | Thời lượng thực thi ngắn |
| Cách chạy | Liên tục | Theo yêu cầu |
| Mở rộng | Cần can thiệp để thêm/bớt máy chủ | Tự động |

- **Định giá:** trả theo số request và thời gian tính toán; slide nêu Free Tier gồm **1.000.000 requests** và **400.000 GB-seconds**.
- Tích hợp với nhiều dịch vụ AWS và ngôn ngữ lập trình; hỗ trợ giám sát qua CloudWatch.
- Có thể cấp đến **10 GB RAM** cho một function; tăng RAM cũng tăng tài nguyên CPU và mạng.

### Ngôn ngữ và loại gói triển khai (Trang 544)

- Ngôn ngữ/runtime được hỗ trợ trong slide: Node.js (JavaScript), Python, Java, C# (.NET Core), PowerShell, Ruby và Custom Runtime API, ví dụ Rust hoặc Golang.
- Có thể triển khai Container Image; image phải triển khai Lambda Runtime API.
- Slide khuyến nghị ECS/Fargate nếu cần chạy Docker image tùy ý.

### Tích hợp Lambda và ví dụ (Trang 545-547)

> *Sơ đồ (Trang 545): Các dịch vụ có thể tích hợp với Lambda.*
>
> *Sơ đồ (Trang 546): Quy trình tạo thumbnail serverless sau khi có ảnh mới.*
>
> *Sơ đồ (Trang 547): EventBridge/CloudWatch Events kích hoạt Lambda theo lịch, ví dụ mỗi giờ.*

### Định giá Lambda (Trang 548)

- Tính phí theo số lần gọi: **1.000.000 requests đầu miễn phí**, sau đó slide nêu **0,20 USD mỗi 1 triệu requests**.
- Tính phí theo thời lượng, làm tròn theo bước **1 ms**.
- Free Tier gồm **400.000 GB-seconds/tháng**; tương đương 400.000 giây ở RAM 1 GB hoặc 3.200.000 giây ở RAM 128 MB.
- Slide nêu mức phí sau Free Tier là **1 USD cho 600.000 GB-seconds**.

## Kiểu gọi Lambda và tích hợp sự kiện (Trang 549-570)

### Gọi đồng bộ (Trang 549-555)

- Các cách gọi đồng bộ gồm CLI, SDK, API Gateway và Application Load Balancer; kết quả được trả ngay.
- Client chịu trách nhiệm xử lý lỗi như retry và exponential backoff.
- Các bên gọi đồng bộ khác trong slide: CloudFront qua Lambda@Edge, S3 Batch, Cognito, Step Functions, Lex, Alexa và Kinesis Data Firehose.
- Có thể dùng ALB hoặc API Gateway để cung cấp HTTP(S) endpoint cho Lambda; function phải được đăng ký trong target group của ALB.
- Khi bật multi-value headers trên ALB, HTTP headers và query string parameters có nhiều giá trị được đưa vào Lambda event/response dưới dạng arrays.

> *Sơ đồ (Trang 551-555): Tích hợp ALB với Lambda, chuyển đổi HTTP/JSON, xử lý multi-value headers và quyền gọi function.*

### Gọi bất đồng bộ và sự kiện S3 (Trang 556-560)

- Nguồn gọi bất đồng bộ gồm S3, SNS và CloudWatch Events/EventBridge; sự kiện được đặt vào event queue trước khi Lambda xử lý.
- Khi function lỗi, Lambda thử tổng cộng **3 lần** theo slide: lần đầu và hai lần thử lại, cách nhau 1 phút rồi 2 phút.
- Thiết kế xử lý có tính lũy đẳng vì retry có thể tạo log entries hoặc tác động lặp.
- Có thể cấu hình DLQ bằng SNS hoặc SQS cho xử lý thất bại; cần IAM permissions phù hợp.
- Các nguồn khác trong slide gồm CodeCommit, CodePipeline, CloudWatch Logs, SES, CloudFormation, AWS Config, AWS IoT và AWS IoT Events.
- S3 event notifications hỗ trợ sự kiện như `ObjectCreated`, `ObjectRemoved`, `ObjectRestore` và `Replication`; có thể lọc theo tên object, ví dụ `*.jpg`.
- S3 thường gửi event trong vài giây nhưng đôi khi mất một phút hoặc lâu hơn.
- Nếu có hai lần ghi đồng thời vào cùng object không bật versioning, có thể chỉ nhận một event. Bật versioning nếu cần event cho từng lần ghi thành công.

> *Sơ đồ (Trang 558-560): EventBridge và S3 event notifications gọi Lambda; ví dụ đồng bộ metadata.*

### Event Source Mapping (Trang 561)

- Lambda dùng **Event Source Mapping** với Kinesis Data Streams, SQS/SQS FIFO và DynamoDB Streams.
- Các nguồn này cần được poll; Lambda đọc records từ nguồn rồi được gọi đồng bộ để xử lý.

### Event Object và Context Object (Trang 562-564)

- **Event Object** là tài liệu JSON chứa dữ liệu cho function; có thông tin từ dịch vụ gọi như EventBridge hoặc đối số đầu vào tùy chỉnh. Lambda runtime chuyển dữ liệu này thành object, ví dụ dict trong Python.
- **Context Object** chứa thông tin và thuộc tính về invocation, function và runtime; Lambda truyền object này cho function.
- Ví dụ thuộc tính: `aws_request_id`, `function_name`, `memory_limit_in_mb`.

### Destinations sau invocation (Trang 565)

- Với asynchronous invocation, có thể gửi kết quả thành công hoặc thất bại đến SQS, SNS, Lambda hoặc EventBridge bus.
- Slide khuyến nghị Destinations thay cho DLQ trong nhiều trường hợp; có thể dùng đồng thời cả hai.
- Event Source Mapping có thể gửi các event batches bị loại bỏ đến SQS hoặc SNS; cũng có thể cấu hình DLQ trực tiếp trên SQS.

### Execution Role và Resource-based Policies (Trang 566-567)

- **Lambda Execution Role** cấp quyền cho function truy cập dịch vụ/tài nguyên AWS. Policy ví dụ:
  - `AWSLambdaBasicExecutionRole`: ghi logs lên CloudWatch.
  - `AWSLambdaKinesisExecutionRole`: đọc Kinesis.
  - `AWSLambdaDynamoDBExecutionRole`: đọc DynamoDB Streams.
  - `AWSLambdaSQSQueueExecutionRole`: đọc SQS.
  - `AWSLambdaVPCAccessExecutionRole`: triển khai Lambda trong VPC.
  - `AWSXRayDaemonWriteAccess`: gửi trace đến X-Ray.
- Với Event Source Mapping, Lambda dùng execution role để đọc dữ liệu từ nguồn. Slide khuyến nghị tạo một execution role riêng cho mỗi function.
- **Resource-based Policy** cấp quyền cho account hoặc dịch vụ AWS khác gọi Lambda, tương tự bucket policy của S3.
- Một principal được phép gọi Lambda nếu identity policy của principal hoặc resource-based policy của Lambda cấp quyền; dịch vụ S3 gọi Lambda qua resource-based policy.

### Environment Variables, Logs, Metrics và X-Ray (Trang 568-570)

- Environment variable là cặp key/value dạng String, dùng đổi hành vi function mà không sửa mã; Lambda cũng thêm biến hệ thống.
- Có thể mã hóa secrets bằng KMS service key hoặc customer-managed key.
- Lambda execution logs được lưu trong CloudWatch Logs; execution role cần quyền ghi logs.
- CloudWatch Metrics của Lambda gồm Invocations, Duration, Concurrent Executions, Errors, Success Rates, Throttles, Async Delivery Failures và Iterator Age cho Kinesis/DynamoDB Streams.
- Bật **Active Tracing** để dùng X-Ray; Lambda chạy daemon, mã dùng X-Ray SDK và execution role cần `AWSXRayDaemonWriteAccess`.
- Biến môi trường liên quan gồm `_X_AMZN_TRACE_ID`, `AWS_XRAY_CONTEXT_MISSING` (mặc định `LOG_ERROR`) và `AWS_XRAY_DAEMON_ADDRESS`.

## Lambda tại Edge (Trang 571-576)

- Edge functions chạy gần người dùng để giảm độ trễ; CloudFront cung cấp CloudFront Functions và Lambda@Edge, không cần tự quản lý máy chủ.
- Trường hợp sử dụng gồm tùy chỉnh nội dung CDN, bảo mật, định tuyến, SEO, giảm bot, biến đổi ảnh, A/B testing, xác thực và phân tích người dùng.

| Đặc điểm | CloudFront Functions | Lambda@Edge |
|---------|----------------------|-------------|
| Runtime | JavaScript | Node.js hoặc Python |
| Quy mô/tốc độ | Sub-millisecond startup, hàng triệu requests/giây | Hàng nghìn requests/giây theo slide |
| Event points | Viewer Request, Viewer Response | Viewer Request/Response và Origin Request/Response |
| Triển khai | Quản lý trực tiếp trong CloudFront | Tạo function tại `us-east-1`, CloudFront sao chép đến các vị trí |
| Phù hợp | Chuẩn hóa Cache Key, sửa header, URL rewrite/redirect, xác thực request | Thời gian chạy dài hơn, thư viện bên thứ ba, truy cập mạng/tệp hoặc HTTP request body |

> *Sơ đồ (Trang 575): So sánh CloudFront Functions và Lambda@Edge.*

## Lambda trong VPC, cấu hình và lưu trữ (Trang 577-586)

### Truy cập tài nguyên VPC và Internet (Trang 577-579)

- Mặc định Lambda chạy ngoài VPC của người dùng, trong VPC do AWS sở hữu; không thể truy cập trực tiếp RDS, ElastiCache hoặc internal ELB trong VPC của người dùng.
- Để chạy trong VPC, khai báo VPC ID, subnets và Security Groups; Lambda tạo ENI trong các subnets. Cần `AWSLambdaVPCAccessExecutionRole`.
- Lambda trong VPC không tự có Internet access; đặt trong public subnet không tạo public IP hoặc kết nối Internet.
- Private subnet cần NAT Gateway/Instance nếu function cần Internet; có thể dùng VPC Endpoints để truy cập riêng tư đến dịch vụ AWS.
- Theo slide, CloudWatch Logs vẫn hoạt động mà không cần endpoint hoặc NAT Gateway.

### RAM, CPU và Timeout (Trang 580)

- RAM từ **128 MB đến 10 GB**, tăng theo bước **1 MB**; tăng RAM cũng tăng vCPU.
- Ở **1.792 MB**, function có tương đương một vCPU đầy đủ; phía trên mức này có thể có nhiều vCPU (tối đa 6). Mã cần multi-threading để tận dụng nhiều CPU.
- Với ứng dụng bị giới hạn bởi CPU, tăng RAM có thể tăng khả năng tính toán.
- Timeout mặc định **3 giây**, tối đa **900 giây (15 phút)**.

### Execution Context và `/tmp` (Trang 581-583)

- Execution Context là môi trường runtime tạm thời khởi tạo dependencies bên ngoài handler như database, HTTP và SDK clients.
- Lambda có thể giữ context một thời gian và tái sử dụng ở invocation sau để tránh khởi tạo kết nối lại.

> *Sơ đồ (Trang 582): Khởi tạo client/kết nối bên ngoài handler để tái sử dụng execution context.*

- `/tmp` cung cấp dung lượng tạm tối đa **10 GB**; dữ liệu có thể còn khi context bị đóng băng và được tái sử dụng như transient cache/checkpoint.
- Dùng S3 cho lưu trữ lâu dài; cần tự tạo KMS Data Keys nếu muốn mã hóa nội dung trong `/tmp` theo slide.

### Layers và hệ thống tệp (Trang 584-585)

- **Lambda Layers** tách dependencies để tái sử dụng và hỗ trợ custom runtimes như C++/Rust.
- Lambda có thể mount EFS nếu chạy trong VPC; mount vào thư mục cục bộ lúc khởi tạo và phải dùng EFS Access Points.
- Theo slide, cần chú ý giới hạn kết nối EFS: mỗi function instance tạo một kết nối, đồng thời có giới hạn burst.

> *Sơ đồ (Trang 586): Các lựa chọn lưu trữ cho Lambda.*

## Concurrency, throttling và Cold Start (Trang 587-590)

- Giới hạn concurrency mặc định trong slide là **1.000 executions đồng thời**.
- **Reserved concurrency** đặt giới hạn concurrency riêng cho function; invocation vượt giới hạn sẽ bị throttle.
- Throttle behavior:
  - Đồng bộ: trả `ThrottleError` với mã 429.
  - Bất đồng bộ: tự retry rồi có thể chuyển đến DLQ.
- Nếu cần concurrency limit cao hơn, yêu cầu tăng giới hạn qua AWS Support.
- Khi không đủ concurrency để xử lý asynchronous events, request bổ sung bị throttle; với lỗi 429 và lỗi hệ thống 500-series, Lambda đưa event về queue và retry tối đa **6 giờ**, khoảng cách tăng dần từ 1 giây đến tối đa 5 phút.
- **Cold Start** xảy ra khi instance mới tải mã và chạy phần khởi tạo ngoài handler; request đầu tiên có độ trễ cao hơn.
- **Provisioned Concurrency** cấp concurrency trước invocation để tránh cold start; Application Auto Scaling có thể điều chỉnh theo lịch hoặc target utilization.

> *Sơ đồ (Trang 588): Tình huống tranh chấp concurrency khi không dành trước giới hạn cho function.*
