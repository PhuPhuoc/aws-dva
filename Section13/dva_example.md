# Câu hỏi ôn tập DVA-C02 - Section 13: Amazon API Gateway

> *Câu hỏi và lựa chọn bằng tiếng Anh; phần giải thích bằng tiếng Việt.*

---

## 1. Chọn HTTP API hay REST API

### Question 1:
A team needs a low-latency API that integrates with Lambda, supports built-in CORS and OAuth 2.0 authorization, and does not require usage plans or API keys. Which API type should the team choose?

```
A. REST API
B. HTTP API
C. Private API only
D. WebSocket API
```

### ✅ Answer: **B. HTTP API**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Theo slide, HTTP API hướng đến độ trễ và chi phí thấp, hỗ trợ Lambda proxy, CORS tích hợp và OIDC/OAuth 2.0.
- Đề bài không cần usage plan hay API key, nên không cần chọn REST API chỉ để có thêm các tính năng đó.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | REST API phù hợp khi cần bộ tính năng nâng cao như usage plan/API key, nhưng không phải lựa chọn tối ưu theo yêu cầu đã nêu. |
| C | Private API mô tả phạm vi truy cập mạng, không phải loại API được yêu cầu để có các tính năng này. |
| D | WebSocket API dành cho giao tiếp hai chiều qua kết nối lâu dài, không phải yêu cầu HTTP API thông thường. |

### 📝 Vocabulary Analysis:
- **low-latency**: có độ trễ thấp.
- **built-in**: được tích hợp sẵn.
- **usage plan**: kế hoạch giới hạn và đo lường mức sử dụng API.

### 🔄 Context Variations:
- **If the API requires usage plans and API keys** → Chọn REST API theo nội dung slide.
- **If clients need persistent, two-way communication** → Xem xét WebSocket API.

---

## 2. Lambda proxy integration

### Question 2:
An API Gateway endpoint invokes Lambda. The function must receive the client request data and return the HTTP status code, headers, and body. Which integration type requires no mapping template and lets the function handle the request and response logic?

```
A. AWS_PROXY
B. MOCK
C. HTTP_PROXY
D. AWS service integration
```

### ✅ Answer: **A. AWS_PROXY**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- `AWS_PROXY` chuyển dữ liệu request vào Lambda theo payload sự kiện; Lambda tự xử lý logic và tạo response.
- Không cần cấu hình mapping template cho từng header hay query string parameter.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | `MOCK` trả response tại API Gateway mà không gọi Lambda/backend. |
| C | `HTTP_PROXY` chuyển tiếp request tới backend HTTP, không phải integration proxy dành cho Lambda. |
| D | AWS service integration gọi API của dịch vụ AWS; không mô tả Lambda proxy request/response. |

### 📝 Vocabulary Analysis:
- **status code**: mã trạng thái HTTP trong response.
- **mapping template**: mẫu ánh xạ để biến đổi dữ liệu request/response.
- **payload**: dữ liệu được truyền trong request hoặc event.

### 🔄 Context Variations:
- **If the request must be transformed before Lambda receives it** → Dùng integration có mapping template phù hợp.
- **If the backend is an HTTP endpoint and should receive the raw HTTP request** → Dùng `HTTP_PROXY`.

---

## 3. Biến đổi JSON sang SOAP/XML

### Question 3:
A company exposes a legacy SOAP service through API Gateway. The client sends JSON, but the backend requires an XML SOAP message. Which API Gateway feature should transform the request and response formats?

```
A. Stage variables
B. API keys
C. Mapping templates
D. Cache invalidation
```

### ✅ Answer: **C. Mapping templates**

### 📖 Explanation (Vietnamese):

**Tại sao chọn C:**
- Mapping template có thể trích xuất dữ liệu từ request, tạo SOAP/XML để gọi backend, rồi biến đổi XML response thành JSON.
- Cách này đặt chuyển đổi giao thức ở lớp API Gateway thay vì yêu cầu client và SOAP service dùng cùng định dạng.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Stage variables chọn giá trị cấu hình theo stage, không biến đổi cấu trúc JSON/XML. |
| B | API key nhận diện client và phục vụ usage plan, không ánh xạ payload. |
| D | Cache invalidation xóa dữ liệu cache, không thay đổi định dạng request/response. |

### 📝 Vocabulary Analysis:
- **legacy**: hệ thống cũ vẫn đang được sử dụng.
- **SOAP service**: dịch vụ dùng giao thức SOAP, thường trao đổi thông điệp XML.
- **transform**: chuyển đổi dữ liệu từ định dạng này sang định dạng khác.

### 🔄 Context Variations:
- **If Lambda can parse JSON and construct the SOAP request itself** → Có thể dùng Lambda proxy và đặt logic chuyển đổi trong function.
- **If the backend accepts the same HTTP payload as the client** → Dùng `HTTP_PROXY` để chuyển tiếp thay vì ánh xạ.

---

## 4. Nhập đặc tả OpenAPI

### Question 4:
A team maintains an API definition as YAML and wants API Gateway to create methods and integrations from that definition. Which approach should the team use?

```
A. Import an OpenAPI specification
B. Create an API key for every method
C. Enable X-Ray tracing
D. Configure a canary deployment
```

### ✅ Answer: **A. Import an OpenAPI specification**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- API Gateway có thể nhập OpenAPI 3.0 để cấu hình method, request, integration và response.
- OpenAPI được viết bằng YAML hoặc JSON và giúp quản lý định nghĩa API dưới dạng mã cấu hình.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | API key kiểm soát/đo lường client, không tạo cấu trúc API từ đặc tả. |
| C | X-Ray thu thập trace để quan sát request, không nhập API definition. |
| D | Canary chuyển một phần traffic sang deployment mới, không tạo method/integration từ YAML. |

### 📝 Vocabulary Analysis:
- **specification**: đặc tả mô tả cấu trúc và hành vi API.
- **as code**: được quản lý bằng tệp cấu hình có thể review và version control.
- **integration**: cấu hình kết nối method API với backend.

### 🔄 Context Variations:
- **If the current API must be documented or reused elsewhere** → Có thể export API thành OpenAPI spec.
- **If an application needs a generated client library** → Dùng OpenAPI để tạo SDK.

---

## 5. Đưa thay đổi lên stage

### Question 5:
An engineer changes a method in API Gateway, but callers still receive the old behavior from the `prod` stage. What must the engineer do for the change to take effect?

```
A. Create another API key
B. Create a new deployment and associate it with the stage
C. Enable CORS on the client
D. Increase the cache capacity
```

### ✅ Answer: **B. Create a new deployment and associate it with the stage**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Thay đổi cấu hình chưa có hiệu lực cho tới khi tạo deployment và đưa deployment đó vào stage.
- Stage `prod` có thể vẫn trỏ đến deployment cũ dù cấu hình mới đã được lưu.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | API key điều khiển nhận diện/usage plan, không cập nhật cấu hình method đang chạy. |
| C | CORS ảnh hưởng request từ trình duyệt khác domain, không triển khai method mới. |
| D | Tăng cache capacity không thay đổi deployment mà stage đang sử dụng. |

### 📝 Vocabulary Analysis:
- **deployment**: phiên bản cấu hình API đã triển khai.
- **stage**: tên môi trường API trỏ tới một deployment, ví dụ `prod`.
- **take effect**: bắt đầu có hiệu lực.

### 🔄 Context Variations:
- **If only the `test` stage should receive the change** → Tạo deployment rồi gắn vào `test`, chưa đổi `prod`.
- **If the new deployment causes errors** → Có thể rollback stage về deployment trước đó.

---

## 6. Stage variables và Lambda alias

### Question 6:
The `dev` and `prod` stages must invoke different Lambda aliases without duplicating the API configuration. Which feature can provide the stage-specific alias value?

```
A. Stage variables
B. API keys
C. CORS headers
D. Usage quotas
```

### ✅ Answer: **A. Stage variables**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Stage variable có thể chứa tên Lambda alias hoặc cấu hình khác theo môi trường.
- API Gateway dùng giá trị tương ứng với stage để gọi đúng alias; cú pháp tham chiếu trong cấu hình là `${stageVariables.variableName}`.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | API key nhận diện client, không chọn Lambda alias theo stage. |
| C | CORS điều khiển việc trình duyệt cho phép gọi API, không chọn backend. |
| D | Quota giới hạn số request, không lưu cấu hình endpoint/alias. |

### 📝 Vocabulary Analysis:
- **alias**: tên tham chiếu đến một phiên bản Lambda.
- **stage-specific**: có giá trị riêng theo từng stage.
- **duplicate**: tạo bản sao cấu hình hoặc tài nguyên.

### 🔄 Context Variations:
- **If `dev` and `prod` use different HTTP backends** → Có thể dùng stage variable cho HTTP endpoint.
- **If the value is a password or token** → Không lưu trong stage variable; dùng Secrets Manager hoặc Parameter Store.

---

## 7. Canary deployment

### Question 7:
A team wants to send 5% of production API traffic to a new deployment, monitor its results separately, and keep the remaining traffic on the current deployment. Which API Gateway feature should it use?

```
A. A usage plan
B. A private endpoint
C. A request validator
D. A canary deployment
```

### ✅ Answer: **D. A canary deployment**

### 📖 Explanation (Vietnamese):

**Tại sao chọn D:**
- Canary deployment cho phép chọn tỷ lệ traffic chuyển tới bản mới.
- Metrics và logs riêng giúp so sánh canary với phần traffic còn lại trước khi phát hành rộng hơn.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Usage plan giới hạn/đo lường mức truy cập của client, không phân traffic giữa deployment. |
| B | Private endpoint giới hạn nguồn mạng có thể truy cập API. |
| C | Request validator kiểm tra dữ liệu request trước integration. |

### 📝 Vocabulary Analysis:
- **canary deployment**: phát hành bản mới cho một phần traffic trước.
- **remaining traffic**: phần lưu lượng còn lại chưa chuyển sang bản mới.
- **monitor separately**: theo dõi metrics/logs độc lập.

### 🔄 Context Variations:
- **If all traffic should move at once** → Triển khai toàn bộ traffic theo deployment mới.
- **If the canary shows a regression** → Dừng tăng traffic và rollback theo kết quả deployment.

---

## 8. API chỉ truy cập từ VPC

### Question 8:
An internal API must be reachable only by workloads in a VPC through an interface endpoint. Which API Gateway endpoint type should be configured?

```
A. Edge-Optimized
B. Private
C. Regional
D. WebSocket
```

### ✅ Answer: **B. Private**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Private API chỉ truy cập từ VPC thông qua interface VPC Endpoint (ENI).
- Resource policy có thể giới hạn quyền truy cập qua VPC Endpoint được phép.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Edge-Optimized định tuyến qua CloudFront Edge Locations cho client toàn cầu. |
| C | Regional dành cho client trong một Region nhưng không giới hạn truy cập chỉ qua VPC Endpoint. |
| D | WebSocket mô tả giao thức giao tiếp hai chiều, không phải endpoint riêng tư được yêu cầu. |

### 📝 Vocabulary Analysis:
- **reachable only**: chỉ có thể truy cập từ phạm vi được chỉ định.
- **interface endpoint**: điểm cuối mạng dạng interface trong VPC để truy cập dịch vụ.
- **resource policy**: policy gắn với tài nguyên để điều khiển principal được phép truy cập.

### 🔄 Context Variations:
- **If clients are public and global** → Xem xét Edge-Optimized.
- **If clients are public but concentrated in one Region** → Xem xét Regional.

---

## 9. Xác thực người dùng bằng Cognito

### Question 9:
A mobile application has external users who sign in through a managed user pool. API Gateway should validate their identity without custom authentication code. Which option is the best fit?

```
A. An API key only
B. An IAM execution role
C. A Cognito User Pool authorizer
D. A MOCK integration
```

### ✅ Answer: **C. A Cognito User Pool authorizer**

### 📖 Explanation (Vietnamese):

**Tại sao chọn C:**
- Cognito User Pool quản lý vòng đời người dùng và token; API Gateway xác minh danh tính từ Cognito.
- Phù hợp người dùng ứng dụng bên ngoài, không cần tự xây dựng logic xác thực token trong Lambda.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | API key hỗ trợ nhận diện/đo lường client, không thay thế xác thực danh tính người dùng. |
| B | IAM execution role cấp quyền cho mã Lambda gọi dịch vụ AWS, không xác thực người dùng mobile. |
| D | MOCK trả response mà không gọi backend, không xác minh danh tính. |

### 📝 Vocabulary Analysis:
- **managed user pool**: kho danh tính người dùng được dịch vụ quản lý.
- **validate identity**: xác minh danh tính người gọi.
- **custom authentication code**: mã xác thực do đội phát triển tự viết và vận hành.

### 🔄 Context Variations:
- **If callers are AWS roles or users signing requests with SigV4** → Dùng IAM authorization.
- **If the application needs to authorize access to a particular record** → Backend vẫn cần kiểm tra quyền theo user và tài nguyên.

---

## 10. Lambda authorizer cho token bên thứ ba

### Question 10:
An API accepts bearer tokens issued by a third-party identity provider. The authorization decision requires custom logic, and API Gateway should receive an IAM policy from that logic. Which mechanism should be used?

```
A. A Lambda authorizer
B. A usage plan
C. A stage variable
D. A cache cluster
```

### ✅ Answer: **A. A Lambda authorizer**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Lambda authorizer có thể nhận bearer token hoặc request parameter, chạy logic xác minh riêng rồi trả IAM policy cho API Gateway.
- API Gateway có thể cache kết quả policy để giảm số lần gọi Lambda authorizer.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | Usage plan áp dụng throttling/quota theo API key, không xác minh token bên thứ ba. |
| C | Stage variable chứa cấu hình theo stage, không đưa ra quyết định phân quyền. |
| D | Cache response của API không thay thế authorizer. |

### 📝 Vocabulary Analysis:
- **bearer token**: token mà người giữ token có thể dùng để gửi request.
- **third-party identity provider**: dịch vụ danh tính bên ngoài hệ thống AWS của ứng dụng.
- **authorization decision**: quyết định cho phép hay từ chối thao tác.

### 🔄 Context Variations:
- **If users authenticate through a Cognito User Pool** → Cấu hình Cognito authorizer.
- **If the caller is an AWS principal signing with SigV4** → Dùng IAM authorization.

---

## 11. IAM authorization và SigV4

### Question 11:
An EC2-based application in the same AWS account calls an API Gateway method. The request must be authenticated using its IAM role credentials. Which authorization method should be configured?

```
A. IAM authorization with Signature Version 4
B. A Cognito User Pool only
C. An API key without an authorizer
D. CORS preflight
```

### ✅ Answer: **A. IAM authorization with Signature Version 4**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Ứng dụng EC2 có IAM role và có thể ký request bằng SigV4.
- IAM policy quyết định quyền gọi method; resource policy có thể bổ sung giới hạn nguồn hoặc hỗ trợ truy cập cross-account.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | Cognito User Pool dành cho danh tính người dùng ứng dụng, không tận dụng IAM role của EC2. |
| C | API key không xác thực IAM principal và không phải cơ chế phân quyền an toàn. |
| D | CORS điều khiển request cross-origin của trình duyệt, không xác thực workload EC2. |

### 📝 Vocabulary Analysis:
- **IAM role credentials**: thông tin xác thực tạm thời gắn với IAM role.
- **Signature Version 4 (SigV4)**: cách ký request AWS bằng credentials.
- **cross-account**: truy cập giữa các AWS account khác nhau.

### 🔄 Context Variations:
- **If a specific VPC Endpoint or source IP must also be enforced** → Bổ sung resource policy phù hợp.
- **If the caller is an external mobile user** → Dùng Cognito hoặc cơ chế authorizer phù hợp thay vì IAM role EC2.

---

## 12. API key và usage plan

### Question 12:
A company sells API access to customers and needs to identify each client, apply throttling and quota limits, and meter usage. Which configuration is appropriate?

```
A. Enable CORS and create one stage
B. Create a Lambda authorizer for every request
C. Use API keys with a usage plan
D. Configure a private endpoint
```

### ✅ Answer: **C. Use API keys with a usage plan**

### 📖 Explanation (Vietnamese):

**Tại sao chọn C:**
- API key nhận diện client; usage plan liên kết API stage, key và giới hạn throttling/quota.
- Cần cấu hình method yêu cầu API key, deploy API, tạo key, tạo usage plan rồi liên kết stage và key; client gửi key trong header `x-api-key`.
- API key phục vụ đo lường/giới hạn, không nên dùng thay authentication và authorization.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | CORS và stage không nhận diện từng client hoặc áp dụng usage quota. |
| B | Lambda authorizer có thể xác thực/phân quyền tùy biến nhưng không tự tạo usage plan theo client. |
| D | Private endpoint giới hạn đường mạng truy cập, không quản lý quota theo customer. |

### 📝 Vocabulary Analysis:
- **meter usage**: ghi nhận mức sử dụng của từng client.
- **throttling**: giới hạn tốc độ request.
- **quota**: giới hạn tổng lượng request trong một khoảng thời gian.

### 🔄 Context Variations:
- **If the API returns user-specific data** → Bổ sung IAM, Cognito hoặc authorizer; không dựa riêng vào API key.
- **If callers must provide an assigned key** → Gửi key bằng header `x-api-key`.

---

## 13. Request validation

### Question 13:
A REST API should reject requests with a missing required query parameter or a payload that does not match the configured JSON Schema before invoking the backend. What should be enabled?

```
A. Response caching
B. Request validation
C. X-Ray tracing
D. Canary deployment
```

### ✅ Answer: **B. Request validation**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Request validation kiểm tra tham số bắt buộc và payload theo JSON Schema trước integration request.
- Request không hợp lệ bị API Gateway từ chối với lỗi 400, tránh gọi backend không cần thiết.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Cache giảm request lặp tới backend nhưng không xác nhận cấu trúc request đầu vào. |
| C | X-Ray giúp tracing, không từ chối payload sai trước integration. |
| D | Canary phân phối một phần traffic tới deployment mới, không xác thực dữ liệu request. |

### 📝 Vocabulary Analysis:
- **required query parameter**: tham số query bắt buộc phải có.
- **payload**: nội dung request được gửi tới API.
- **JSON Schema**: định nghĩa cấu trúc và kiểu dữ liệu JSON hợp lệ.

### 🔄 Context Variations:
- **If the request is structurally valid but violates a business rule** → Backend vẫn phải thực hiện kiểm tra nghiệp vụ.
- **If the validation fails** → API Gateway trả 400 trước khi gọi integration.

---

## 14. Cache response API

### Question 14:
A production REST API repeatedly returns the same public response. The team wants to reduce backend calls and knows the response can be stale for up to five minutes. Which configuration best meets the requirement?

```
A. Enable API Gateway caching with a 300-second TTL
B. Create an API key with a 300-second quota
C. Enable a canary deployment for every request
D. Use a private API endpoint
```

### ✅ Answer: **A. Enable API Gateway caching with a 300-second TTL**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Cache response giảm số lần gọi backend; TTL 300 giây là giá trị mặc định được slide nêu.
- Vì response là dữ liệu công khai và có thể cũ tối đa năm phút, cache phù hợp với yêu cầu.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | API key và quota không lưu response để giảm backend calls. |
| C | Canary kiểm soát traffic phát hành, không cache response. |
| D | Private endpoint giới hạn nguồn truy cập, không làm giảm request lặp. |

### 📝 Vocabulary Analysis:
- **stale**: không còn phản ánh dữ liệu mới nhất.
- **TTL (time to live)**: thời gian cache giữ response trước khi hết hạn.
- **backend call**: request API Gateway gửi tới dịch vụ phía sau.

### 🔄 Context Variations:
- **If the response contains private user-specific data** → Tắt cache hoặc thiết kế cache key/authorization để tách biệt an toàn.
- **If the data must always be current** → Không dùng cache hoặc đặt TTL phù hợp với yêu cầu freshness.

---

## 15. Bảo vệ thao tác invalidate cache

### Question 15:
Clients must not be able to flush the API Gateway cache unless they have explicit IAM authorization. Which configuration should be applied?

```
A. Require authorization for cache invalidation
B. Add an API key to the response body
C. Enable a regional endpoint
D. Increase the cache TTL
```

### ✅ Answer: **A. Require authorization for cache invalidation**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Client có thể yêu cầu invalidate cache bằng `Cache-Control: max-age=0`.
- Cần bật yêu cầu authorization hoặc áp dụng policy `InvalidateCache`; nếu không, client có thể làm trống cache và tăng tải backend.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | Đưa API key vào response body không kiểm soát quyền invalidate. |
| C | Regional endpoint xác định kiểu endpoint, không cấp quyền thao tác cache. |
| D | TTL dài hơn không ngăn client gửi yêu cầu invalidate. |

### 📝 Vocabulary Analysis:
- **flush/invalidate the cache**: xóa cache để request sau lấy dữ liệu mới.
- **explicit authorization**: quyền được cấp rõ ràng cho principal.
- **cache invalidation**: thao tác làm dữ liệu cache hết hiệu lực.

### 🔄 Context Variations:
- **If no client should invalidate the cache** → Không bật quyền invalidation cho các method/client đó.
- **If an operator needs to clear stale data** → Cấp IAM quyền tối thiểu cho operator thực hiện.

---

## 16. Phân biệt Latency và IntegrationLatency

### Question 16:
CloudWatch shows high API `Latency` but low `IntegrationLatency`. Which conclusion is most appropriate?

```
A. The backend is definitely the only slow component
B. Time is being spent outside the measured backend integration
C. The API has no client requests
D. The cache must be disabled
```

### ✅ Answer: **B. Time is being spent outside the measured backend integration**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- `Latency` tính thời gian từ khi API Gateway nhận request đến khi trả response.
- `IntegrationLatency` chỉ đo thời gian từ lúc chuyển request tới backend đến lúc nhận response. Vì vậy, `Latency` cao nhưng `IntegrationLatency` thấp gợi ý phần thời gian còn lại nằm ngoài integration.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | IntegrationLatency thấp không cho thấy backend là thành phần chậm duy nhất. |
| C | Latency chỉ được đo khi có request; kết quả không có nghĩa API không nhận request. |
| D | Hai metrics không đủ để kết luận cache phải bị tắt. |

### 📝 Vocabulary Analysis:
- **measured interval**: khoảng thời gian được metric đo.
- **integration**: phần kết nối và xử lý tại backend.
- **overhead**: thời gian xử lý bổ sung ngoài backend.

### 🔄 Context Variations:
- **If both Latency and IntegrationLatency are high** → Điều tra backend và dependency phía sau.
- **If CacheHitCount is low and requests repeat** → Kiểm tra cấu hình cache và cache key.

---

## 17. Xử lý throttling

### Question 17:
A caller receives `429 Too Many Requests` from API Gateway. The backend is being overloaded by repeated immediate retries. What should the client do?

```
A. Retry every request immediately without a limit
B. Ignore the response and continue sending requests
C. Retry with exponential backoff and jitter
D. Replace the API key on every request
```

### ✅ Answer: **C. Retry with exponential backoff and jitter**

### 📖 Explanation (Vietnamese):

**Tại sao chọn C:**
- `429` báo request bị throttling; đây là lỗi có thể retry nhưng cần giãn khoảng cách giữa các lần thử.
- Exponential backoff và jitter giảm khả năng nhiều client retry đồng thời, tránh làm tải tăng thêm.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Retry tức thì có thể tạo vòng lặp throttling và tiếp tục làm quá tải backend. |
| B | Tiếp tục gửi request không xử lý nguyên nhân bị giới hạn. |
| D | Đổi API key không giải quyết giới hạn stage/account và có thể phá usage metering. |

### 📝 Vocabulary Analysis:
- **throttling**: giới hạn tốc độ request do vượt ngưỡng.
- **exponential backoff**: tăng dần thời gian chờ giữa các lần retry.
- **jitter**: thêm độ ngẫu nhiên vào thời gian chờ để tránh retry đồng loạt.

### 🔄 Context Variations:
- **If one API is starving other APIs in the account** → Đặt giới hạn phù hợp ở stage/method và bảo vệ backend.
- **If the caller receives a non-retriable 4xx error** → Sửa request hoặc quyền thay vì retry.

---

## 18. Phân biệt lỗi 502 và 504

### Question 18:
A Lambda proxy integration returns a response that does not follow the expected format, while another request times out waiting for the backend. Which API Gateway errors correspond to these cases?

```
A. 400 and 403
B. 429 and 503
C. 503 and 502
D. 502 and 504
```

### ✅ Answer: **D. 502 and 504**

### 📖 Explanation (Vietnamese):

**Tại sao chọn D:**
- `502 Bad Gateway` thường xuất hiện khi output từ Lambda proxy không tương thích định dạng API Gateway mong đợi.
- `504 Integration Failure` có thể xuất hiện khi integration/backend timeout.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | `400` và `403` là lỗi request/quyền phía client, không mô tả response proxy sai hoặc timeout backend. |
| B | `429` là throttling; `503` là service unavailable, không phải cặp tình huống đã nêu. |
| C | Cả hai mã không khớp thứ tự tình huống trong đề. |

### 📝 Vocabulary Analysis:
- **incompatible format**: định dạng response không tương thích với hợp đồng yêu cầu.
- **integration failure**: lỗi khi API Gateway giao tiếp với backend.
- **time out**: hết thời gian chờ mà chưa nhận được response.

### 🔄 Context Variations:
- **If the caller sends an invalid request** → Có thể nhận lỗi 400.
- **If the request exceeds a throttling limit** → Có thể nhận 429.

---

## 19. CORS và preflight request

### Question 19:
A browser application hosted on a different domain calls an API Gateway endpoint. The browser sends an `OPTIONS` preflight request. Which response headers must be configured to allow the cross-origin request?

```
A. Access-Control-Allow-Methods, Access-Control-Allow-Headers, and Access-Control-Allow-Origin
B. Cache-Control, ETag, and Last-Modified
C. x-api-key, Authorization, and Content-Length only
D. X-Ray-Trace-Id and CloudWatch-Log-Group
```

### ✅ Answer: **A. Access-Control-Allow-Methods, Access-Control-Allow-Headers, and Access-Control-Allow-Origin**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- CORS cho phép trình duyệt gọi API từ domain khác theo origin, method và header được chấp thuận.
- OPTIONS preflight cần các header `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers` và `Access-Control-Allow-Origin`.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | Đây là các header liên quan cache HTTP, không cấp phép cross-origin. |
| C | API key/Authorization có thể là header request nhưng không thay thế response header CORS. |
| D | Trace/log không điều khiển chính sách CORS của trình duyệt. |

### 📝 Vocabulary Analysis:
- **cross-origin**: request từ trang có domain khác với domain API.
- **preflight request**: request OPTIONS mà trình duyệt gửi trước một số request cross-origin.
- **allow-origin**: header chỉ định origin được phép gọi API.

### 🔄 Context Variations:
- **If the browser preflight succeeds but the actual call is blocked** → Kiểm tra CORS headers trên response thực tế, không chỉ OPTIONS.
- **If callers are backend services rather than browsers** → CORS không phải cơ chế xác thực hay kiểm soát truy cập cho các caller đó.
