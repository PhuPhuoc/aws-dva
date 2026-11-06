# Section 13 - Amazon API Gateway (Trang 663-700)

---

## Tổng quan về API Gateway (Trang 663-668)

### Khả năng của API Gateway (Trang 664)

- Kết hợp Lambda với API Gateway để xây dựng API mà không cần quản lý hạ tầng máy chủ.
- Hỗ trợ giao thức WebSocket.
- Quản lý phiên bản API, ví dụ v1 và v2.
- Quản lý nhiều môi trường như dev, test và prod.
- Hỗ trợ xác thực và phân quyền.
- Tạo API key và kiểm soát giới hạn request.
- Nhập định nghĩa Swagger/OpenAPI để nhanh chóng xác định API.
- Chuyển đổi và xác thực request/response.
- Tạo SDK và đặc tả API.
- Lưu cache cho response API.

### HTTP API và REST API (Trang 665)

| Loại API | Đặc điểm theo slide |
|---|---|
| HTTP API | Độ trễ thấp, chi phí thấp; hỗ trợ Lambda proxy, HTTP proxy và tích hợp riêng tư; không có ánh xạ dữ liệu; hỗ trợ OIDC, OAuth 2.0 và CORS tích hợp; không có usage plan và API key. |
| REST API | Có đầy đủ tính năng của API Gateway, ngoại trừ hỗ trợ OIDC/OAuth 2.0 tích hợp sẵn. |

### Các loại tích hợp (Trang 666)

- **Lambda:** gọi Lambda để cung cấp REST API phía sau.
- **HTTP:** đưa endpoint HTTP phía sau lên API; ví dụ API nội bộ tại chỗ hoặc Application Load Balancer. API Gateway có thể bổ sung giới hạn request, cache, xác thực người dùng và API key.
- **AWS Service:** đưa API của dịch vụ AWS qua API Gateway; ví dụ khởi chạy quy trình Step Functions hoặc gửi message vào SQS. API Gateway có thể bổ sung xác thực, triển khai API công khai và kiểm soát tốc độ.

### Kiến trúc API Gateway (Trang 663, 667-668)

- Tạo một giao diện chung cho các microservice trong tổ chức.
- API có thể chứa nhiều endpoint và resource.
- Có thể dùng domain name đơn giản cùng SSL certificate.
- Có thể chuyển tiếp và biến đổi request ở API Gateway.

> *Sơ đồ (Trang 663): Ví dụ xây dựng API serverless.*
>
> *Sơ đồ (Trang 668): API Gateway tích hợp với Kinesis Data Streams.*

## Endpoint và bảo mật domain (Trang 669-670)

### Các loại endpoint (Trang 669)

- **Edge-Optimized (mặc định):** dành cho client toàn cầu; request đi qua CloudFront Edge Locations để giảm độ trễ. API Gateway vẫn thuộc một Region.
- **Regional:** dành cho client trong cùng Region. Có thể tự kết hợp với CloudFront để kiểm soát chiến lược cache và distribution.
- **Private:** chỉ truy cập được từ VPC thông qua interface VPC Endpoint (ENI); dùng resource policy để quy định quyền truy cập.

### Xác thực và domain tùy chỉnh (Trang 670)

- Các lựa chọn xác thực người dùng:
  - IAM Role, phù hợp với ứng dụng nội bộ.
  - Cognito, phù hợp với người dùng bên ngoài, ví dụ ứng dụng di động.
  - Custom Authorizer, dùng logic riêng.
- Bảo vệ custom domain bằng HTTPS thông qua AWS Certificate Manager (ACM):
  - Với endpoint Edge-Optimized, certificate phải ở `us-east-1`.
  - Với endpoint Regional, certificate phải ở Region của API Gateway.
- Cấu hình bản ghi CNAME hoặc A-alias trong Route 53.

## Deployment, stage và phiên bản (Trang 671-675)

### Deployment và stage (Trang 671-672)

- Thay đổi cấu hình trong API Gateway chưa có hiệu lực cho đến khi tạo deployment.
- Deployment được đưa vào một stage; có thể tạo nhiều stage như dev, test, prod.
- Mỗi stage có các tham số cấu hình riêng.
- API Gateway lưu lịch sử deployment để có thể rollback.
- Có thể duy trì endpoint phiên bản như `/v1` và `/v2` khi có thay đổi breaking change.

### Stage variables (Trang 673-674)

- Stage variables tương tự environment variables của API Gateway, dùng cho các giá trị cấu hình thường xuyên thay đổi.
- Có thể dùng trong Lambda function ARN, HTTP endpoint và mapping template.
- Ví dụ: chọn HTTP endpoint cho từng môi trường hoặc truyền tham số cấu hình đến Lambda qua mapping template.
- Stage variables được đưa vào object `context` của Lambda; cú pháp: `${stageVariables.variableName}`.
- Có thể dùng stage variable để chọn Lambda alias tương ứng với stage.

### Canary deployment (Trang 675)

- Có thể bật canary deployment cho bất kỳ stage nào, thường là prod.
- Chọn tỷ lệ traffic chuyển tới canary channel.
- Metrics và logs của canary được tách riêng để theo dõi.
- Có thể override stage variables cho canary.
- Đây là mô hình blue/green deployment với Lambda và API Gateway.

## Loại tích hợp và mapping template (Trang 676-682)

### Các loại integration (Trang 676-678)

- **MOCK:** API Gateway trả response mà không gửi request tới backend.
- **HTTP/AWS (Lambda và AWS Services):** cần cấu hình integration request và integration response; có thể dùng mapping template để ánh xạ dữ liệu.
- **AWS_PROXY (Lambda Proxy):** request từ client được chuyển đến Lambda; function xử lý logic request/response. Không cần mapping template; headers và query string được chuyển trong dữ liệu request.
- **HTTP_PROXY:** chuyển HTTP request tới backend và trả response backend qua API Gateway; không cần mapping template. Có thể thêm HTTP header, ví dụ API key.

### Mapping template (Trang 679-681)

- Dùng để sửa đổi request/response, đổi tên hoặc biến đổi query string parameter, chỉnh body, thêm header và lọc bỏ dữ liệu không cần thiết.
- Dùng Velocity Template Language (VTL), hỗ trợ cấu trúc như vòng lặp và điều kiện.
- Content-Type có thể là `application/json` hoặc `application/xml`.
- Với SOAP dùng XML còn REST API thường dùng JSON; API Gateway có thể:
  1. Trích xuất dữ liệu từ path, payload hoặc header của request.
  2. Dùng mapping template tạo SOAP message.
  3. Gọi SOAP service và nhận XML response.
  4. Chuyển XML response sang định dạng mong muốn, ví dụ JSON, rồi trả cho client.

> *Sơ đồ (Trang 681): Ví dụ ánh xạ query string parameter.*

### OpenAPI (Trang 682)

- OpenAPI là cách phổ biến để định nghĩa REST API dưới dạng mã cấu hình.
- Có thể nhập OpenAPI 3.0 vào API Gateway để tạo cấu hình method, method request, integration request và method response; có thể bổ sung AWS extension cho các tùy chọn riêng của API Gateway.
- Có thể xuất API hiện tại thành OpenAPI.
- Định nghĩa OpenAPI có thể viết bằng YAML hoặc JSON và dùng để tạo SDK cho ứng dụng.

## Xác thực request, cache và usage plan (Trang 683-688)

### Xác thực request (Trang 683-684)

- REST API có thể xác thực cơ bản request trước khi gửi integration request.
- Nếu xác thực thất bại, API Gateway dừng xử lý và trả lỗi 400; nhờ đó giảm request không cần thiết tới backend.
- Có thể kiểm tra:
  - URI parameter, query string và header bắt buộc có được gửi và không để trống hay không.
  - Payload có tuân theo JSON Schema request model của method hay không.
- Có thể cấu hình request validation bằng cách nhập OpenAPI definition.

### Cache response (Trang 685-686)

- Cache giảm số lần gọi backend.
- TTL mặc định là 300 giây; phạm vi cấu hình theo slide là 0-3600 giây.
- Cache được cấu hình theo stage; có thể override theo từng method.
- Có tùy chọn mã hóa cache; dung lượng cache trong slide là 0,5-237 GB.
- Cache tốn chi phí, thường phù hợp môi trường production hơn dev/test.
- Có thể xóa toàn bộ cache ngay lập tức.
- Client có thể yêu cầu xóa cache bằng header `Cache-Control: max-age=0` nếu có IAM authorization phù hợp.
- Nếu không áp dụng policy `InvalidateCache` hoặc không bật yêu cầu authorization, mọi client đều có thể yêu cầu xóa cache API.

### Usage plan và API key (Trang 687-688)

- Usage plan quy định ai có thể truy cập một hay nhiều API stage/method, tần suất và tổng lượng request.
- API key là chuỗi chữ-số cấp cho client để nhận diện và đo lường mức sử dụng; có thể kết hợp với usage plan.
- Usage plan cấu hình giới hạn throttling và quota cho từng client.
- Trình tự cấu hình:
  1. Tạo API, cấu hình method yêu cầu API key và triển khai API lên stage.
  2. Tạo hoặc nhập API key rồi cấp cho client.
  3. Tạo usage plan với giới hạn throttling và quota.
  4. Gắn API stage và API key vào usage plan.
- Client gửi API key trong header `x-api-key`.

## Logging, metrics, throttling, lỗi và CORS (Trang 689-695)

### Logging và tracing (Trang 689)

- CloudWatch Logs có thể ghi thông tin request/response body.
- Bật CloudWatch logging ở cấp stage, chọn log level ERROR, DEBUG hoặc INFO; có thể override theo API.
- Bật X-Ray tracing để xem thêm thông tin request; kết hợp API Gateway và Lambda để có góc nhìn xuyên suốt.

### CloudWatch metrics (Trang 690)

- Metrics được thu thập theo stage; có thể bật detailed metrics.
- `CacheHitCount` và `CacheMissCount`: hiệu quả cache.
- `Count`: tổng số API request trong khoảng thời gian.
- `IntegrationLatency`: thời gian từ lúc API Gateway chuyển request đến backend đến lúc nhận response.
- `Latency`: thời gian từ lúc API Gateway nhận request từ client đến lúc trả response; gồm `IntegrationLatency` và phần overhead của API Gateway.
- `4XXError` là lỗi phía client; `5XXError` là lỗi phía server.

### Throttling (Trang 691)

- Slide nêu account limit là 10.000 request/giây trên toàn bộ API; đây là soft limit và có thể yêu cầu tăng.
- Khi bị throttling, API Gateway trả `429 Too Many Requests`; đây là lỗi có thể retry.
- Có thể đặt giới hạn ở stage và method để điều chỉnh tải.
- Usage plan có thể áp dụng throttling theo client.
- Nếu không giới hạn, một API quá tải có thể khiến các API khác trong cùng account bị throttling, tương tự tranh chấp concurrency ở Lambda.

### Mã lỗi (Trang 692)

- **4xx:** lỗi phía client.
  - `400 Bad Request`.
  - `403 Access Denied`, có thể do WAF lọc request.
  - `429 Quota exceeded/Throttle`.
- **5xx:** lỗi phía server.
  - `502 Bad Gateway`: thường do output không tương thích từ backend tích hợp Lambda proxy; đôi khi do invocation sai thứ tự khi tải cao.
  - `503 Service Unavailable`.
  - `504 Integration Failure`: ví dụ endpoint request timeout. Slide nêu request API Gateway timeout tối đa sau 29 giây.

### CORS (Trang 693-694)

- Cần bật CORS khi API nhận request từ domain khác.
- OPTIONS preflight request cần các header:
  - `Access-Control-Allow-Methods`
  - `Access-Control-Allow-Headers`
  - `Access-Control-Allow-Origin`
- Có thể bật CORS qua Console.

> *Sơ đồ (Trang 694): Cấu hình CORS trên API Gateway.*

## Các lựa chọn bảo mật (Trang 695-699)

### IAM và resource policy (Trang 695-696)

- Tạo IAM policy cấp cho User hoặc Role để xác thực và phân quyền; phù hợp với user/role trong AWS, ví dụ EC2, Lambda và IAM user.
- IAM dùng khả năng SigV4; thông tin xác thực được ký trong request header.
- Resource policy có thể:
  - Cho phép truy cập cross-account khi kết hợp với IAM security.
  - Giới hạn theo source IP.
  - Giới hạn theo VPC Endpoint.

### Cognito User Pool (Trang 697)

- Cognito quản lý vòng đời người dùng; token tự hết hạn.
- API Gateway tự xác minh danh tính từ Cognito, không cần tự viết phần xác thực.
- Xác thực do Cognito User Pool đảm nhiệm; API Gateway có thể phân quyền ở cấp method, còn backend cần kiểm tra quyền nghiệp vụ/trên từng tài nguyên khi cần.

### Lambda Authorizer (Trang 698)

- Token-based authorizer nhận bearer token, ví dụ JWT hoặc OAuth.
- Request parameter-based Lambda authorizer có thể dùng header, query string hoặc stage variable.
- Lambda trả IAM policy cho user; kết quả policy được cache.

### Tóm tắt các cơ chế bảo mật (Trang 699)

- **IAM:** phù hợp user/role trong AWS; kết hợp resource policy cho cross-account; xử lý cả xác thực và phân quyền bằng Signature v4.
- **Custom Authorizer:** phù hợp token bên thứ ba; Lambda xác minh xác thực và trả phân quyền linh hoạt; tính phí theo lần gọi Lambda và có cache kết quả.
- **Cognito User Pool:** quản lý user pool, có thể kết hợp đăng nhập Facebook/Google; không cần viết mã xác thực riêng nhưng backend cần thực hiện phân quyền.

## Bài tập trên slide (Trang 700)

- Xây dựng API Gateway tích hợp Lambda để kết nối DynamoDB.
