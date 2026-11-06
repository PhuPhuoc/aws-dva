# Giải thích bổ sung - Section 13: Amazon API Gateway

> *Phần này bổ sung kinh nghiệm triển khai; nội dung mở rộng được tách khỏi phần dịch slide.*

---

### 1. Chọn HTTP API hay REST API

#### Giải thích cốt lõi

HTTP API và REST API đều đưa API ra cho client, nhưng khác nhau về phạm vi tính năng. Theo slide, HTTP API ưu tiên độ trễ và chi phí thấp, phù hợp tích hợp proxy với Lambda/HTTP, hỗ trợ OIDC/OAuth 2.0 và CORS; đổi lại, không có usage plan và API key. REST API có nhiều khả năng cấu hình hơn như request validation, mapping template, cache và usage plan. Vì vậy, chọn loại API theo tính năng thực sự cần, không chỉ theo tên giao thức.

#### NÊN làm (DO)

- **Bắt đầu với HTTP API khi tích hợp proxy và các tính năng sẵn có đáp ứng yêu cầu** → Mô hình gọn hơn, độ trễ và chi phí thường thấp hơn; so với REST API, tránh trả thêm cho các tính năng không dùng.
- **Chọn REST API khi cần các tính năng nâng cao mà HTTP API không cung cấp trong hợp đồng hiện tại** → Ví dụ mapping phức tạp, cache, usage plan hoặc API key theo nội dung slide; so với việc tự dựng lớp điều khiển riêng, dùng đúng loại API giảm vận hành phụ.
- **Đối chiếu tính năng và giới hạn theo loại API trước khi thiết kế** → API Gateway có nhiều loại endpoint, integration và authorizer; so với giả định mọi loại API có cùng khả năng, kiểm tra sớm tránh phải chuyển đổi sau này.

#### KHÔNG NÊN làm (DON'T)

- **Chọn REST API chỉ vì tên có chữ REST** → Có thể trả chi phí và chịu thêm cấu hình dù chỉ cần proxy đơn giản. NÊN: chọn HTTP API nếu các tính năng cần thiết đã được đáp ứng.
- **Giả định HTTP API và REST API có cùng usage plan, cache, mapping và validation** → Cấu hình cần thiết có thể không tồn tại trên loại API đã chọn. NÊN: lập danh sách tính năng bắt buộc rồi kiểm tra tương thích trước khi triển khai.

---

### 2. Chọn integration và nơi đặt logic biến đổi

#### Giải thích cốt lõi

`AWS_PROXY` chuyển request vào Lambda theo dạng payload sự kiện; Lambda chịu trách nhiệm diễn giải dữ liệu và tạo response HTTP. `HTTP_PROXY` chuyển tiếp request đến backend HTTP rồi chuyển response trở lại. Các integration `HTTP/AWS` không dùng proxy cần cấu hình integration request/response và có thể dùng mapping template. `MOCK` trả response ngay tại API Gateway mà không gọi backend. Proxy giảm cấu hình trung gian; integration có mapping phù hợp khi cần biến đổi giao thức hoặc tương thích backend cũ.

#### NÊN làm (DO)

- **Dùng Lambda proxy cho API mới nếu Lambda có thể tự xử lý request và response** → Ít cấu hình ánh xạ và giữ logic ở mã ứng dụng; so với mapping template phức tạp, dễ kiểm thử và bảo trì hơn.
- **Dùng mapping template khi cần chuyển đổi định dạng hoặc tích hợp backend cũ** → Ví dụ đổi JSON sang SOAP/XML; so với sửa backend không thể thay đổi, API Gateway có thể làm lớp chuyển đổi biên.
- **Dùng MOCK cho response cố định hoặc tình huống không cần backend** → Không tạo request mạng tới dịch vụ khác; so với gọi backend chỉ để trả dữ liệu tĩnh, giảm độ trễ và chi phí.

#### KHÔNG NÊN làm (DON'T)

- **Cố gắn mapping template vào Lambda proxy hoặc HTTP proxy như thể chúng là integration thường** → Proxy chuyển payload theo cơ chế riêng, cấu hình ánh xạ không được dùng như mong đợi. NÊN: xử lý payload trong Lambda/backend hoặc chuyển sang integration có mapping.
- **Đặt nghiệp vụ phức tạp trong VTL** → Logic khó kiểm thử độc lập và khó theo dõi lỗi. NÊN: giữ template cho biến đổi giao thức/dữ liệu; đặt quyết định nghiệp vụ trong backend.

---

### 3. Deployment, stage và phát hành canary

#### Giải thích cốt lõi

Thay đổi cấu hình API Gateway chưa có hiệu lực cho tới khi tạo deployment. Deployment là phiên bản cấu hình được gắn vào stage như dev, test hoặc prod; mỗi stage có cấu hình riêng và có thể trỏ tới deployment khác nhau. Stage variables giúp chọn endpoint hoặc Lambda alias theo môi trường. Canary gửi một tỷ lệ request sang cấu hình mới, cho phép quan sát riêng metrics/logs trước khi tăng traffic hoặc rollback.

#### NÊN làm (DO)

- **Tự động hóa deployment sau mỗi thay đổi API cần đưa vào sử dụng** → Tránh nhầm cấu hình trên Console với cấu hình đang chạy; so với thao tác thủ công, pipeline giúp môi trường test/prod nhất quán.
- **Dùng stage variables cho cấu hình môi trường không nhạy cảm và dùng alias để chọn phiên bản Lambda** → Có thể chuyển stage giữa endpoint/alias mà không nhân bản toàn bộ API; so với hard-code ARN hoặc URL, thay đổi môi trường rõ ràng hơn.
- **Tăng dần tỷ lệ canary và theo dõi metrics riêng trước khi chuyển toàn bộ traffic** → Giảm phạm vi ảnh hưởng nếu bản mới lỗi; so với phát hành đồng loạt, canary cho tín hiệu rollback sớm.
- **Dùng version path như `/v1` và `/v2` khi thay đổi hợp đồng không tương thích** → Client cũ tiếp tục hoạt động trong thời gian chuyển đổi; so với sửa endpoint đang dùng, giảm nguy cơ phá vỡ tích hợp.

#### KHÔNG NÊN làm (DON'T)

- **Cho rằng lưu thay đổi trên API Gateway đồng nghĩa đã cập nhật stage** → Stage vẫn có thể chạy deployment cũ. NÊN: tạo deployment và xác nhận stage trỏ đúng deployment.
- **Lưu password, token hoặc secret trong stage variable** → Giá trị cấu hình không phải kho secret và có thể bị lộ qua quyền đọc cấu hình/log. NÊN: lưu secret trong Secrets Manager hoặc Parameter Store với IAM hạn chế.
- **Chuyển 100% traffic ngay khi chưa quan sát bản mới** → Lỗi có thể ảnh hưởng toàn bộ người dùng trước khi được phát hiện. NÊN: dùng canary hoặc chiến lược phát hành từng bước khi cần giảm rủi ro.

---

### 4. Chọn endpoint và cấu hình custom domain

#### Giải thích cốt lõi

Edge-Optimized định tuyến request qua CloudFront Edge Locations để phục vụ client phân tán toàn cầu, nhưng API Gateway vẫn nằm trong một Region. Regional phù hợp client tập trung trong một Region; có thể ghép CloudFront riêng khi cần kiểm soát cache. Private chỉ cho phép truy cập qua interface VPC Endpoint và resource policy. Với custom domain, vị trí ACM certificate phụ thuộc endpoint: `us-east-1` cho Edge-Optimized và Region của API cho Regional.

#### NÊN làm (DO)

- **Chọn Regional khi client chủ yếu ở cùng Region** → Đường đi ngắn và kiểm soát CloudFront riêng nếu cần; so với Edge-Optimized mặc định, tránh thêm lớp phân phối không cần thiết.
- **Dùng Private endpoint cùng VPC Endpoint và resource policy cho API nội bộ** → Lưu lượng không cần đi qua endpoint công khai; so với public endpoint có IP allowlist, giảm bề mặt truy cập.
- **Kiểm tra Region của ACM certificate và DNS record khi tạo custom domain** → Certificate sai Region khiến cấu hình domain thất bại; so với xử lý lỗi sau khi triển khai, kiểm tra trước rút ngắn thời gian phát hành.

#### KHÔNG NÊN làm (DON'T)

- **Coi Edge-Optimized là cơ chế triển khai đa Region hoặc tự tạo khả năng failover giữa Region** → API vẫn ở một Region. NÊN: thiết kế replication/failover riêng nếu yêu cầu chịu lỗi đa Region.
- **Dùng Private endpoint cho client Internet công cộng** → Client ngoài VPC không có đường truy cập trực tiếp qua interface endpoint. NÊN: chọn Regional/Edge-Optimized hoặc thiết kế kết nối mạng phù hợp.

---

### 5. Phân biệt xác thực, phân quyền và API key

#### Giải thích cốt lõi

IAM xác thực request bằng SigV4 và cấp quyền theo IAM policy; resource policy trên API Gateway giới hạn principal, account, IP hoặc VPC Endpoint được phép truy cập. Cognito User Pool quản lý danh tính/token người dùng; API Gateway xác minh token, còn ứng dụng/backend vẫn cần quyết định người dùng được phép làm gì. Lambda authorizer kiểm tra bearer token hoặc request parameter rồi trả IAM policy; kết quả có thể được cache. API key và usage plan dùng để nhận diện/đo lường client và đặt giới hạn, không thay thế cơ chế xác thực người dùng.

#### NÊN làm (DO)

- **Dùng IAM/SigV4 cho workload hoặc người dùng đã có danh tính AWS** → Quyền gắn với principal và IAM policy; so với tự quản lý token riêng cho ứng dụng nội bộ, tận dụng danh tính và quyền AWS sẵn có.
- **Dùng Cognito User Pool cho đăng nhập người dùng ứng dụng và Lambda authorizer khi cần quy tắc token riêng** → Mỗi cơ chế xử lý loại danh tính phù hợp; so với tự viết toàn bộ xác thực trong backend, giảm phần mã phải tự vận hành.
- **Kết hợp API key/usage plan với xác thực phù hợp nếu API có dữ liệu cần bảo vệ** → API key giúp đo lường và giới hạn client, còn IAM/Cognito/authorizer quyết định danh tính và quyền; so với chỉ dựa vào chuỗi API key, có ranh giới bảo mật rõ hơn.
- **Giới hạn quyền resource policy theo account, nguồn IP hoặc VPC Endpoint khi phù hợp** → Giảm các nguồn có thể gọi API; so với chỉ mở quyền rộng rồi dựa vào kiểm tra phía ứng dụng, chặn được request sớm hơn.

#### KHÔNG NÊN làm (DON'T)

- **Coi API key là thông tin xác thực an toàn hoặc cơ chế phân quyền người dùng** → Key có thể bị chia sẻ/lộ và chỉ phục vụ nhận diện, đo lường theo usage plan. NÊN: dùng IAM, Cognito hoặc authorizer để xác thực/phân quyền.
- **Bật cache kết quả Lambda authorizer mà không xét cách policy được tái sử dụng** → Quyết định đã cache có thể áp dụng cho request khác cùng cache key. NÊN: thiết kế identity source/cache key và policy sao cho mọi request được cache đều có cùng phạm vi quyền.
- **Chỉ xác minh token rồi bỏ qua kiểm tra quyền trên tài nguyên ở backend** → Người dùng đã đăng nhập vẫn có thể không được phép thao tác trên mọi bản ghi. NÊN: backend kiểm tra quyền theo user/resource sau khi xác thực.

---

### 6. OpenAPI, request validation và CORS

#### Giải thích cốt lõi

OpenAPI mô tả hợp đồng API dưới dạng YAML/JSON, có thể đưa vào kiểm soát phiên bản và nhập vào API Gateway. Request validation kiểm tra cấu trúc cơ bản như tham số bắt buộc và payload theo JSON Schema trước khi gọi backend; nó không thay thế kiểm tra nghiệp vụ. CORS là quy tắc trình duyệt cho phép trang web ở domain khác gọi API; preflight `OPTIONS` hỏi trước method/header nào được phép, còn response thực tế cũng phải trả cấu hình phù hợp.

#### NÊN làm (DO)

- **Quản lý OpenAPI spec cùng mã ứng dụng và dùng nó làm hợp đồng giữa client/backend** → Review được thay đổi API và giảm lệch giữa tài liệu với triển khai; so với cấu hình rời rạc trên Console, thay đổi dễ kiểm soát hơn.
- **Bật request validation cho cấu trúc và trường bắt buộc có thể kiểm tra ở biên** → Request sai bị trả 400 trước khi gọi backend; so với chỉ phát hiện lỗi trong ứng dụng, tiết kiệm invocation và tài nguyên phía sau.
- **Cấu hình CORS theo đúng origin, method và header cần thiết** → Trình duyệt chỉ cho phép luồng ứng dụng dự kiến; so với wildcard không cần thiết, giới hạn được các trang web có thể gọi từ trình duyệt.
- **Giữ kiểm tra nghiệp vụ và quyền truy cập trong backend** → JSON Schema không biết trạng thái tài khoản hay quyền trên từng bản ghi; so với chỉ dựa vào validation ở gateway, bảo vệ nghiệp vụ vẫn đúng ở mọi đường gọi.

#### KHÔNG NÊN làm (DON'T)

- **Coi request validation hoặc CORS là cơ chế phân quyền** → Validation chỉ kiểm tra hình dạng dữ liệu; CORS chủ yếu được trình duyệt áp dụng và không chặn client không phải trình duyệt. NÊN: xác thực và kiểm tra quyền độc lập.
- **Chỉ thêm header CORS cho OPTIONS mà quên response thật** → Preflight có thể thành công nhưng trình duyệt vẫn chặn response API. NÊN: cấu hình header cần thiết cho cả preflight và response tích hợp.

---

### 7. Cache, usage plan và giới hạn tải

#### Giải thích cốt lõi

Cache response ở API Gateway giảm request tới backend nhưng có thể trả dữ liệu cũ; cache key phải phân biệt các đầu vào làm thay đổi response. Usage plan đặt ngưỡng throttling/quota theo client có API key. Giới hạn account, stage và method bảo vệ API Gateway/backend ở các cấp khác nhau; vượt ngưỡng thường trả 429. Các quota/throttling trong usage plan là mục tiêu kiểm soát lưu lượng, không nên xem là hàng rào bảo mật tuyệt đối.

#### NÊN làm (DO)

- **Chỉ cache response có thể chia sẻ an toàn và có độ mới phù hợp** → Cache giảm tải khi nhiều request lặp lại; so với cache mọi response, tránh trả nhầm dữ liệu cá nhân hoặc dữ liệu theo quyền người dùng.
- **Đưa query/header ảnh hưởng response vào cache key và giới hạn quyền invalidate** → Mỗi biến thể có cache riêng và client không thể tùy ý làm rỗng cache; so với cache key quá rộng, giảm rò dữ liệu và giảm chi phí do invalidation.
- **Đặt throttling ở stage/method và phía backend cho các luồng quan trọng** → Bảo vệ dependency ngay cả khi lưu lượng vượt giới hạn ở gateway; so với chỉ dựa vào usage plan, có nhiều lớp giới hạn hơn.
- **Retry 429 có exponential backoff và jitter** → Giãn các lần thử lại để tránh dồn thêm tải; so với retry liên tục, giảm nguy cơ tạo vòng lặp throttling.

#### KHÔNG NÊN làm (DON'T)

- **Cache response theo user mà không đưa yếu tố danh tính/quyền vào chiến lược cache** → Client khác có thể nhận response không thuộc về họ. NÊN: tắt cache cho dữ liệu riêng tư hoặc thiết kế cache key/authorization đúng.
- **Dùng API key/usage plan làm giới hạn cứng để bảo vệ backend hoặc tính phí chính xác** → Throttling và quota có thể không phải ranh giới tuyệt đối. NÊN: bổ sung kiểm soát trong ứng dụng/backend và theo dõi lưu lượng thực tế.
- **Cho mọi client quyền invalidate cache** → Client có thể liên tục làm giảm cache hit và tăng tải/chi phí backend. NÊN: yêu cầu IAM authorization cho invalidation và giới hạn principal được phép.

---

### 8. Logging, metrics và xử lý lỗi

#### Giải thích cốt lõi

`Latency` đo toàn bộ thời gian từ khi API Gateway nhận request tới khi trả response; `IntegrationLatency` chỉ đo phần chờ backend. Chênh lệch lớn giữa hai metric gợi ý thời gian ở gateway hoặc bước xử lý ngoài integration. CloudWatch Logs và X-Ray giúp điều tra request, còn mã 4xx thường chỉ ra request/quyền phía client và 5xx thường chỉ ra lỗi gateway/backend. Request body trong logs có thể chứa dữ liệu nhạy cảm.

#### NÊN làm (DO)

- **Theo dõi `4XXError`, `5XXError`, `Latency`, `IntegrationLatency` và cache hit/miss theo stage** → Phân biệt lỗi client, backend và hiệu quả cache; so với chỉ nhìn tổng số request, khoanh vùng sự cố nhanh hơn.
- **Dùng X-Ray khi cần lần theo request qua API Gateway và Lambda** → Có thể nhìn các đoạn xử lý nối tiếp; so với chỉ dùng log rời, dễ xác định dependency chậm.
- **Kiểm tra response format của Lambda proxy khi gặp 502** → Backend có thể trả cấu trúc không tương thích; so với retry gateway, sửa response contract giải quyết nguyên nhân.
- **Dùng xử lý bất đồng bộ hoặc hàng đợi cho công việc vượt thời gian request phù hợp** → Client không phải giữ kết nối trong khi tác vụ dài; so với kéo dài request đồng bộ, giảm timeout và retry không cần thiết.
- **Kiểm tra quota/timeout hiện hành theo loại API và Region** → Các con số như account RPS hoặc timeout có thể phụ thuộc quota và cấu hình; so với hard-code số liệu từ slide, xác minh trước khi đặt SLO.

#### KHÔNG NÊN làm (DON'T)

- **Ghi toàn bộ request/response body mà không xem xét dữ liệu nhạy cảm** → Log có thể lưu token, thông tin cá nhân hoặc dữ liệu nghiệp vụ; NÊN: giảm nội dung log, che dữ liệu và giới hạn quyền truy cập CloudWatch Logs.
- **Retry mọi lỗi 4xx như lỗi tạm thời** → Request sai hoặc thiếu quyền thường không tự hết khi retry. NÊN: sửa request/quyền; chỉ retry lỗi throttling hoặc lỗi tạm thời với backoff.
- **Cho rằng 29 giây hoặc 10.000 request/giây luôn là giới hạn bất biến cho mọi cấu hình** → Quota phụ thuộc loại API, Region và cấu hình tài khoản. NÊN: kiểm tra quota hiện hành trước khi dựa vào các con số trong slide.
