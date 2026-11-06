# Giải thích bổ sung - Section 11: AWS Lambda

> *Phần này bổ sung kiến thức triển khai; nội dung mở rộng được tách khỏi phần dịch slide.*

---

### 1. Chọn Lambda thay cho máy chủ luôn chạy

#### Giải thích cốt lõi

Lambda chạy function theo invocation mà không yêu cầu người dùng quản lý máy chủ. Đây là mô hình phù hợp với công việc ngắn, được kích hoạt bởi request hoặc sự kiện và có thể tự mở rộng. EC2 phù hợp hơn khi ứng dụng cần tiến trình chạy liên tục, quyền kiểm soát môi trường máy chủ hoặc thời gian xử lý vượt giới hạn Lambda.

Chi phí Lambda phụ thuộc số lần gọi và thời gian/tài nguyên tính toán; EC2 tính phí cho máy chủ được cấp phát theo thời gian chạy. Tăng RAM Lambda cũng tăng CPU/mạng, nên cần đo tải thay vì chỉ tăng tài nguyên tùy tiện.

#### NÊN làm (DO)

- **Dùng Lambda cho tác vụ ngắn, kích hoạt theo sự kiện và không cần tiến trình liên tục** → Không cần tự quản lý fleet máy chủ và Lambda tự tăng/giảm số invocation. So với EC2 chạy thường trực cho tải thưa, có thể tránh trả phí cho thời gian máy chủ nhàn rỗi.
- **Đo Duration, Errors, Throttles và mức RAM trước khi chọn cấu hình** → Mức RAM ảnh hưởng cả CPU/mạng và thời gian chạy. So với chọn cấu hình theo phỏng đoán, đo tải giúp cân bằng chi phí và hiệu năng.
- **Chọn ECS/Fargate hoặc EC2 khi cần chạy Docker tùy ý hay workload dài liên tục** → Lambda Container Image vẫn phải tuân theo Lambda Runtime API và timeout. So với ép workload dài vào function ngắn hạn, compute liên tục phù hợp hơn.

#### KHÔNG NÊN làm (DON'T)

- **Xem Serverless là không có máy chủ hoặc không có giới hạn vận hành** → AWS vẫn vận hành server; function vẫn có giới hạn concurrency, timeout và tài nguyên. NÊN: thiết kế theo giới hạn Lambda và dùng compute khác khi workload không phù hợp.
- **Chuyển một workload luôn chạy sang Lambda mà không phân tích cách tính phí và thời gian thực thi** → Invocation dài/lặp liên tục có thể không phù hợp với mô hình tính theo thời lượng. NÊN: so sánh với EC2/ECS/Fargate dựa trên tải thật.

---

### 2. Phân biệt invocation và Event Source Mapping

#### Giải thích cốt lõi

Invocation đồng bộ trả kết quả cho bên gọi ngay; bên gọi chịu trách nhiệm retry và backoff khi lỗi. Invocation bất đồng bộ nhận sự kiện vào hàng đợi nội bộ rồi xử lý sau, không yêu cầu bên gửi chờ kết quả. Event Source Mapping là cơ chế Lambda poll những nguồn như SQS, Kinesis Data Streams và DynamoDB Streams; Lambda nhận records và được gọi đồng bộ để xử lý batch.

S3 gọi Lambda theo cơ chế bất đồng bộ, không phải Event Source Mapping. Phân biệt cách nguồn giao sự kiện rất quan trọng vì nó quyết định ai poll dữ liệu, retry ở đâu và lỗi được xử lý thế nào.

#### NÊN làm (DO)

- **Dùng đồng bộ khi client cần kết quả ngay, ví dụ API Gateway gọi Lambda** → Client nhận response trực tiếp và có thể áp dụng retry/backoff theo hợp đồng API. So với bất đồng bộ, không cần hàng đợi phía sau khi phản hồi là một phần của request.
- **Dùng bất đồng bộ cho sự kiện như S3/SNS khi bên gửi không cần chờ kết quả** → Lambda nhận sự kiện qua event queue và tiếp tục xử lý riêng. So với đồng bộ, bên phát sự kiện không phải giữ kết nối chờ function hoàn tất.
- **Dùng Event Source Mapping cho SQS, Kinesis Data Streams hoặc DynamoDB Streams** → Lambda poll nguồn rồi gọi function với records. So với S3 notification, các nguồn này cần consumer đọc/poll theo vị trí dữ liệu.

#### KHÔNG NÊN làm (DON'T)

- **Coi S3 notification là Event Source Mapping** → S3 gửi asynchronous invocation; cấu hình nhầm cơ chế sẽ làm sai mô hình retry và quyền truy cập. NÊN: cấu hình S3 làm nguồn gọi bất đồng bộ, dùng Event Source Mapping cho SQS/Kinesis/DynamoDB Streams.
- **Dùng invocation đồng bộ cho công việc dài mà người gọi không cần chờ** → Kết nối phía gọi bị giữ mở và lỗi/retry phụ thuộc client. NÊN: dùng bất đồng bộ hoặc hàng đợi phù hợp.

---

### 3. Retry, DLQ/Destinations và tính lũy đẳng

#### Giải thích cốt lõi

Invocation bất đồng bộ được Lambda retry khi function lỗi; slide nêu tổng cộng ba lần thử và khoảng chờ một phút rồi hai phút. Nếu function có side effects, một lần retry có thể lặp lại thao tác đã hoàn tất trước khi lỗi được báo. Event Source Mapping có quy tắc retry riêng theo nguồn; batch bị loại bỏ có thể chuyển đến SQS/SNS.

DLQ giữ invocation thất bại để xử lý sau; Destinations có thể nhận kết quả thành công hoặc thất bại của invocation bất đồng bộ. Chúng khác nhau về dữ liệu/kết quả gửi đi và nguồn lỗi, nên phải chọn theo mục tiêu vận hành.

#### NÊN làm (DO)

- **Thiết kế handler bất đồng bộ có tính lũy đẳng** → Sự kiện có thể được xử lý lại, vì vậy cùng một event không nên tạo side effect lặp. So với giả định mỗi event chỉ chạy một lần, thao tác lũy đẳng an toàn hơn khi retry.
- **Cấu hình DLQ hoặc Destination để quan sát và xử lý lỗi cuối cùng** → Nếu không có nơi nhận, sự kiện lỗi khó điều tra sau khi hết retry. So với chỉ dựa vào logs, hàng đợi/destination giữ được payload hoặc kết quả cần xử lý tiếp.
- **Kiểm tra IAM permissions cho DLQ/Destinations và nguồn gọi** → Lambda phải có quyền gửi dữ liệu đến đích. So với chỉ bật cấu hình trên Console, kiểm tra quyền giúp tránh lỗi gửi thất bại ở thời điểm runtime.
- **Theo dõi Async Delivery Failures và Iterator Age khi phù hợp** → Hai metrics chỉ ra thất bại giao bất đồng bộ hoặc độ trễ đọc stream. So với chỉ theo dõi Invocations, chúng phản ánh tình trạng backlog tốt hơn.

#### KHÔNG NÊN làm (DON'T)

- **Giả định async invocation chỉ chạy một lần hoặc không thể tạo duplicate** → Retry có thể ghi nhiều log entries hoặc lặp side effect. NÊN: dùng idempotency key/trạng thái xử lý và kiểm tra duplicate.
- **Cho rằng Event Source Mapping dùng cùng cơ chế retry như S3 async invocation** → Event Source Mapping poll batch và xử lý lỗi theo nguồn, khác với hàng đợi invocation bất đồng bộ. NÊN: cấu hình retry/DLQ/destination theo loại event source.

---

### 4. Execution Role, resource policy và secrets

#### Giải thích cốt lõi

Lambda dùng **Execution Role** để function gọi các dịch vụ AWS; đó là danh tính của mã khi chạy. **Resource-based Policy** trên function cho phép dịch vụ hoặc account khác được gọi function. Hai policy giải quyết hai hướng quyền khác nhau: function truy cập tài nguyên và bên khác gọi function.

Environment variables là cặp key/value dạng String giúp thay đổi cấu hình mà không sửa mã. Mã hóa bằng KMS bảo vệ giá trị lưu trữ, nhưng secret vẫn cần được quản lý theo vòng đời và quyền truy cập phù hợp.

#### NÊN làm (DO)

- **Cấp Execution Role theo đúng dịch vụ function cần gọi** → Ví dụ quyền CloudWatch Logs, DynamoDB hoặc SQS. So với gắn quyền quản trị rộng, role hẹp giảm hậu quả nếu mã bị khai thác.
- **Tạo role riêng cho từng function có quyền khác nhau** → Function chỉ nhận tập quyền cần thiết. So với một role dùng chung, thay đổi quyền ít ảnh hưởng chéo hơn.
- **Dùng resource-based policy để cho S3 hoặc account khác gọi Lambda** → Chính sách nằm trên function, thể hiện rõ ai được phép invoke. So với nhầm lẫn với Execution Role, đây là lớp quyền đúng cho bên gọi.
- **Không đặt secret lâu dài trong plain environment variables** → Mã hóa bằng KMS hoặc lấy secret từ Secrets Manager/SSM SecureString theo nhu cầu. So với ghi password trực tiếp trong cấu hình, cách này giảm rủi ro lộ secret.

#### KHÔNG NÊN làm (DON'T)

- **Dùng Execution Role để cấp quyền cho S3 gọi function** → Execution Role cấp quyền cho mã Lambda đi gọi AWS, không cấp quyền invoke cho dịch vụ gọi. NÊN: thêm resource-based policy cho principal dịch vụ.
- **Gắn AdministratorAccess chỉ để function chạy được** → Nếu function bị khai thác, phạm vi tài nguyên bị ảnh hưởng rất rộng. NÊN: xác định API cần thiết và cấp quyền tối thiểu.
- **Coi mã hóa environment variable là thay thế cho quản lý secret** → Người có quyền đọc cấu hình hoặc mã vẫn có thể tiếp cận secret. NÊN: giới hạn IAM access và dùng dịch vụ quản lý secret khi cần xoay vòng/audit.

---

### 5. VPC, kết nối và EFS

#### Giải thích cốt lõi

Lambda mặc định chạy ngoài VPC của người dùng. Khi cần truy cập tài nguyên riêng như RDS, cấu hình VPC ID, subnets và Security Groups để Lambda tạo ENI trong VPC. Khi Lambda đã gắn vào VPC, Internet access cần đường đi như NAT; đặt function trong public subnet không tự cấp public IP. VPC Endpoint cho phép truy cập riêng tư đến dịch vụ AWS được hỗ trợ.

EFS cho lưu trữ chia sẻ bền vững, nhưng Lambda phải chạy trong VPC và dùng EFS Access Points. Đây khác với `/tmp`, vốn là vùng lưu trữ tạm gắn với execution environment.

#### NÊN làm (DO)

- **Chỉ gắn Lambda vào VPC khi cần truy cập tài nguyên riêng** → Kết nối qua ENI giúp vào RDS/private services, nhưng thêm cấu hình mạng. So với mặc định ngoài VPC, chỉ dùng VPC khi có dependency nội bộ giúp giảm thành phần cần vận hành.
- **Dùng NAT cho Internet egress và VPC Endpoint cho truy cập AWS riêng tư phù hợp** → Public subnet không tự cấp Internet cho Lambda. So với mở đường Internet cho mọi truy cập AWS, endpoint có thể giữ traffic đến dịch vụ trong mạng AWS.
- **Dùng EFS khi cần hệ thống tệp dùng chung và dữ liệu tồn tại lâu hơn invocation** → EFS bền vững hơn `/tmp` nhưng có giới hạn kết nối. So với file tạm, EFS phù hợp dữ liệu chia sẻ; với object lưu lâu dài, S3 thường phù hợp hơn.

#### KHÔNG NÊN làm (DON'T)

- **Cho rằng đặt Lambda vào public subnet sẽ cấp public IP/Internet** → Slide nêu Lambda không nhận public IP chỉ vì subnet public. NÊN: dùng NAT Gateway/Instance cho Internet hoặc VPC Endpoint cho AWS service.
- **Dùng EFS cho mọi dữ liệu tạm của function** → EFS thêm latency, kết nối và giới hạn mount. NÊN: dùng `/tmp` cho dữ liệu tạm cục bộ, S3 cho object bền vững và EFS cho filesystem chia sẻ.

---

### 6. Memory, execution context, `/tmp` và package

#### Giải thích cốt lõi

Lambda cấp RAM từ 128 MB đến 10 GB; tăng RAM đồng thời tăng CPU/network. Execution Context giữ runtime tạm thời và có thể được tái sử dụng, vì vậy khởi tạo SDK/database connection ngoài handler giúp giảm thời gian khởi động ở invocation sau. `/tmp` là cache tạm có thể tồn tại khi context được freeze, nhưng không phải lưu trữ bền vững.

Layers cho phép dùng lại dependencies hoặc custom runtime; timeout giới hạn thời gian tối đa của một invocation. Tối ưu cần cân bằng tốc độ, kích thước package và chi phí.

#### NÊN làm (DO)

- **Khởi tạo SDK clients và kết nối dùng chung bên ngoài handler** → Execution Context có thể được reuse, tránh khởi tạo lại mỗi invocation. So với tạo client trong mỗi lần gọi, cách này giảm công việc lặp.
- **Benchmark RAM/CPU và timeout bằng dữ liệu workload thật** → Tăng RAM có thể tăng CPU, giảm thời gian chạy nhưng thay đổi chi phí. So với luôn cấp tối đa, đo thử giúp chọn cấu hình hợp lý.
- **Dùng `/tmp` cho cache/tệp tạm, S3 cho dữ liệu cần giữ lâu** → `/tmp` chỉ tồn tại cùng execution context, còn S3 là lưu trữ object bền vững. So với dùng `/tmp` như kho dữ liệu, S3 tránh mất dữ liệu khi context bị thay thế.
- **Dùng Layers khi nhiều functions cần chung dependencies/runtime** → Dependencies được tách khỏi deployment package. So với copy cùng thư viện vào từng gói, layer giúp tái sử dụng và cập nhật chung.

#### KHÔNG NÊN làm (DON'T)

- **Tạo database/SDK connection mới trong mọi invocation mà không cần** → Tăng thời gian khởi tạo và số kết nối. NÊN: tái sử dụng client ngoài handler, đồng thời xử lý trường hợp context bị thay thế.
- **Dựa vào execution context hoặc `/tmp` để lưu trạng thái bền vững** → AWS có thể thay instance/context và dữ liệu tạm sẽ mất. NÊN: lưu trạng thái nghiệp vụ ở database/S3 phù hợp.
- **Tăng RAM mà không đo latency/cost** → Có thể trả thêm phí nhưng không cải thiện bottleneck. NÊN: kiểm thử các mức RAM và so sánh thời lượng thực thi.

---

### 7. Concurrency, cold starts và xử lý tại Edge

#### Giải thích cốt lõi

Concurrency là số invocation chạy đồng thời; reserved concurrency đặt giới hạn dành riêng cho function và cũng giới hạn mức tối đa của function đó. Vượt giới hạn gây throttling. **Provisioned Concurrency** cấp sẵn execution environments trước invocation để giảm cold start; đây không giống Reserved Concurrency, vốn không khởi tạo sẵn môi trường.

CloudFront Functions và Lambda@Edge đều chạy gần người dùng nhưng phục vụ mức độ phức tạp khác nhau. CloudFront Functions tối ưu cho thao tác nhẹ, nhanh ở viewer request/response; Lambda@Edge hỗ trợ các event points khác và xử lý nặng hơn nhưng tốn tài nguyên hơn.

#### NÊN làm (DO)

- **Dùng Reserved Concurrency để dành và giới hạn concurrency cho function quan trọng** → Function không bị các function khác lấy hết phần concurrency đã dành; cần đặt giới hạn đủ cao cho peak. So với không giới hạn riêng, cách này cô lập mức sử dụng nhưng không xử lý được tải vượt chính giới hạn đó.
- **Dùng Provisioned Concurrency khi cần latency thấp ổn định ngay từ invocation đầu** → Execution environments được cấp trước. So với chỉ Reserved Concurrency, provisioned concurrency giải quyết cold start nhưng có chi phí cho capacity đã cấp.
- **Dùng CloudFront Functions cho thao tác nhẹ cần chạy nhanh ở viewer request/response** → Thời gian khởi động dưới mili giây theo slide. So với Lambda@Edge, lựa chọn này phù hợp hơn cho chỉnh header, URL và Cache Key đơn giản.
- **Dùng Lambda@Edge khi cần thư viện, thời gian chạy dài hơn, network/filesystem hoặc HTTP body** → Nó hỗ trợ nhiều event points hơn CloudFront Functions. So với function nhẹ ở edge, Lambda@Edge phù hợp logic phức tạp hơn.

#### KHÔNG NÊN làm (DON'T)

- **Cho rằng Reserved Concurrency tự loại bỏ throttling khi tải vượt giới hạn đã đặt** → Invocation vượt limit vẫn bị throttle. NÊN: đặt limit theo peak, xin tăng service quota nếu cần và dùng SQS để hấp thụ burst khi workload phù hợp.
- **Chọn Provisioned Concurrency chỉ để dành quota** → Nó cấp sẵn môi trường và có chi phí; dành quota không đồng nghĩa cần giảm cold start. NÊN: dùng Reserved Concurrency để cô lập mức concurrency, Provisioned Concurrency cho độ trễ khởi động.
- **Đưa logic nặng hoặc truy cập dịch vụ bên ngoài vào CloudFront Functions** → Runtime giới hạn hơn Lambda@Edge. NÊN: chọn Lambda@Edge khi cần networking, file system, body hoặc thư viện bổ sung.
