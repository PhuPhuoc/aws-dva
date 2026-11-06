# Giải thích bổ sung - Section 10: AWS Giám sát, xử lý sự cố và kiểm toán

> *Phần này bổ sung kiến thức thực tế; nội dung mở rộng được tách biệt khỏi phần dịch slide.*

---

### 1. Chọn đúng tín hiệu: metrics, logs, traces và audit events

#### Giải thích cốt lõi

Bốn loại dữ liệu trả lời các câu hỏi khác nhau. **Metrics** là các giá trị tổng hợp theo thời gian, phù hợp phát hiện xu hướng và kích hoạt alarm. **Logs** lưu chi tiết từng sự kiện để tra cứu nguyên nhân. **Traces** nối các bước xử lý của cùng một yêu cầu qua nhiều dịch vụ để xác định nơi phát sinh độ trễ/lỗi. **CloudTrail events** ghi lại ai hoặc dịch vụ nào gọi API, vào lúc nào và thay đổi tài nguyên gì.

CloudWatch tập trung vào tình trạng vận hành của hệ thống; X-Ray tập trung vào đường đi của yêu cầu; CloudTrail tập trung vào hoạt động API và kiểm toán. Chúng bổ sung cho nhau, không thay thế lẫn nhau.

#### NÊN làm (DO)

- **Dùng CloudWatch Metrics và Alarms để theo dõi sức khỏe theo ngưỡng** → Metrics có thể được đánh giá liên tục và kích hoạt hành động. So với tìm thủ công trong logs, alarm phát hiện sớm tình trạng như CPU cao.
- **Dùng Logs để tra sự kiện cụ thể và X-Ray để lần theo yêu cầu qua nhiều dịch vụ** → Logs cho nội dung chi tiết, traces cho quan hệ thời gian/phụ thuộc. So với chỉ có metric tổng hợp, kết hợp hai nguồn giúp xác định thành phần gây lỗi.
- **Dùng CloudTrail khi cần biết principal nào gọi API hoặc thay đổi tài nguyên** → CloudTrail có danh tính và hoạt động API; so với X-Ray, đây là nguồn phù hợp cho kiểm toán thay đổi tài khoản.

#### KHÔNG NÊN làm (DON'T)

- **Dùng một dịch vụ để trả lời mọi câu hỏi vận hành** → Metrics không chứa đầy đủ nội dung từng sự kiện, logs không tự dựng đường đi end-to-end, còn CloudTrail không đo hiệu năng ứng dụng. NÊN: chọn tín hiệu theo câu hỏi và đối chiếu nhiều nguồn khi điều tra.
- **Coi CloudTrail là nơi thay thế application logs hoặc X-Ray** → CloudTrail ghi hoạt động API AWS, không cung cấp stack trace hay thời gian từng bước xử lý ứng dụng. NÊN: dùng CloudWatch Logs và X-Ray cho chẩn đoán ứng dụng.

---

### 2. CloudWatch Metrics, custom metrics và alarms

#### Giải thích cốt lõi

Metric là chuỗi giá trị có namespace, tên, timestamp và dimensions. Namespace phân tách nguồn metrics; dimensions giúp chia dữ liệu theo thuộc tính như instance hoặc môi trường. Standard metrics do dịch vụ phát ra; các giá trị như RAM hoặc chỉ số nghiệp vụ cần được gửi dưới dạng custom metrics. EC2 basic monitoring gửi metric mỗi 5 phút, còn detailed monitoring tăng tần suất lên 1 phút; độ phân giải custom metric cao hơn có thể xuống 1, 5, 10 hoặc 30 giây và tốn thêm chi phí.

Alarm đánh giá metric trong từng period rồi chuyển giữa `OK`, `ALARM` và `INSUFFICIENT_DATA`. Composite alarm tổng hợp trạng thái nhiều alarms bằng điều kiện logic để tránh gửi nhiều thông báo cho cùng một vấn đề. Detailed monitoring tăng độ mới của metric EC2, nhưng không tự tạo metrics về RAM; CloudWatch Agent cần thu thập dữ liệu đó.

#### NÊN làm (DO)

- **Cài CloudWatch Agent khi cần RAM, dung lượng đĩa hoặc chỉ số ứng dụng trên EC2** → Những dữ liệu này không có trong standard metrics mặc định. So với bật detailed monitoring, Agent thu thập loại metric mà basic/detailed EC2 metrics không cung cấp.
- **Chọn period và độ phân giải theo thời gian phản ứng cần thiết** → 1 phút phù hợp hơn 5 phút khi ASG cần phản ứng nhanh; high resolution có thể phát hiện ngắn hơn nhưng tăng chi phí. So với bật high resolution cho mọi metric, giới hạn nó vào luồng cần phản ứng nhanh.
- **Dùng composite alarm khi nhiều điều kiện cùng mô tả một sự cố** → AND/OR có thể giảm alarm noise. So với thông báo riêng cho từng metric, một composite alarm có thể biểu đạt trạng thái tổng hợp rõ hơn.
- **Tạo metric filter trước khi cần alarm cho mẫu log** → Metric filter chỉ phát sinh datapoints cho log events xuất hiện sau khi cấu hình. So với truy vấn lại lịch sử bằng Logs Insights, filter hỗ trợ giám sát liên tục kể từ thời điểm tạo.

#### KHÔNG NÊN làm (DON'T)

- **Giả định EC2 standard metrics có RAM hoặc phần trăm đĩa đã dùng** → Các dữ liệu này không được gửi mặc định, khiến alarm dựa trên chúng không có metric đầu vào. NÊN: cấu hình CloudWatch Agent và kiểm tra IAM Role, kết nối, tệp cấu hình.
- **Bật detailed monitoring với hy vọng nó bổ sung loại metric còn thiếu** → Nó tăng tần suất gửi metric chứ không bổ sung RAM. NÊN: dùng Agent cho dữ liệu hệ điều hành, chỉ bật detailed monitoring khi cần chu kỳ 1 phút.
- **Tạo metric filter sau sự cố rồi mong nó tính lại logs cũ** → Filter không hồi tố, nên metric không thể phản ánh dữ liệu trước lúc tạo. NÊN: dùng Logs Insights để truy vấn lịch sử và giữ metric filter cho giám sát tiếp diễn.

---

### 3. CloudWatch Logs: truy vấn, xuất và subscription

#### Giải thích cốt lõi

Log group gom dữ liệu theo ứng dụng/dịch vụ; log stream là chuỗi sự kiện cụ thể trong group. **Logs Insights** truy vấn dữ liệu đã lưu theo yêu cầu, còn **subscription** chuyển các log events mới gần thời gian thực đến Kinesis Data Streams, Data Firehose hoặc Lambda. **S3 export task** là tác vụ xuất dữ liệu hàng loạt; slide lưu ý dữ liệu có thể mất đến 12 giờ để sẵn sàng, nên không phù hợp với luồng xử lý tức thời.

CloudWatch Agent cần được cài đặt, cấp quyền và cấu hình để gửi logs/metrics từ EC2 hoặc máy chủ tại chỗ. Việc giữ logs trong CloudWatch hay xuất sang S3 phụ thuộc vào nhu cầu truy vấn và thời gian lưu.

#### NÊN làm (DO)

- **Đặt retention policy theo yêu cầu điều tra và tuân thủ** → Log group có thể hết hạn theo cấu hình; nếu không đặt chính sách phù hợp, dữ liệu có thể bị giữ quá lâu hoặc hết hạn trước khi cần. So với giữ mọi logs vô thời hạn trong CloudWatch, đặt thời hạn rồi lưu trữ dài hạn ở S3 khi cần giúp tách nhu cầu truy vấn thường xuyên khỏi lưu trữ dài hạn.
- **Dùng subscription filter cho phân tích hoặc chuyển tiếp log events gần thời gian thực** → Subscription đẩy dữ liệu mới đến đích được hỗ trợ và lọc được log events. So với S3 export task, đây là hướng phù hợp khi pipeline cần dữ liệu mới liên tục.
- **Cấu hình Agent bằng quyền tối thiểu, tệp cấu hình và kết nối đã kiểm tra** → Agent cần IAM permissions và đường truyền đến CloudWatch. So với chỉ cài phần mềm, xác nhận đủ cả ba điều kiện giúp tránh tình trạng agent chạy nhưng không gửi dữ liệu.

#### KHÔNG NÊN làm (DON'T)

- **Dùng S3 export task như pipeline thời gian thực** → Dữ liệu xuất có thể mất đến 12 giờ mới sẵn sàng, khiến xử lý bị trễ. NÊN: dùng Logs Subscriptions đến Kinesis Data Streams, Data Firehose hoặc Lambda.
- **Chỉ bật Logs Insights khi cần alarm liên tục theo mẫu log** → Insights là công cụ truy vấn, không phải cơ chế phát metric liên tục. NÊN: tạo metric filter rồi gắn CloudWatch Alarm; dùng Insights để điều tra dữ liệu đã ghi.
- **Cho rằng EC2 tự gửi application logs đến CloudWatch** → Mặc định logs trên máy không được gửi. NÊN: cài CloudWatch Agent, khai báo tệp cần thu thập và cấp quyền IAM phù hợp.

---

### 4. EventBridge cho phản ứng và phát lại sự kiện

#### Giải thích cốt lõi

EventBridge Rule khớp sự kiện theo lịch hoặc event pattern rồi định tuyến đến target. Event bus là điểm nhận sự kiện; resource-based policy cho phép tài khoản khác gửi/nhận sự kiện theo quyền. Archive giữ sự kiện khớp điều kiện để có thể replay, còn Schema Registry suy ra cấu trúc dữ liệu để ứng dụng tạo mã tương thích.

So với CloudWatch Alarm, EventBridge phản ứng với sự kiện có cấu trúc chứ không đánh giá chuỗi metric theo ngưỡng. So với SQS, archive/replay của EventBridge phục vụ phát lại sự kiện; SQS vẫn phù hợp làm hàng đợi công việc cần consumer nhận và xóa thông điệp.

#### NÊN làm (DO)

- **Dùng event pattern cho thay đổi trạng thái dịch vụ và schedule cho tác vụ định kỳ** → Hai loại rule phản ánh hai điều kiện kích hoạt khác nhau. So với polling API theo chu kỳ, event pattern phản ứng khi sự kiện phù hợp xuất hiện.
- **Bật archive khi cần phát lại sự kiện để phục hồi hoặc kiểm tra xử lý** → Replay dùng dữ liệu đã lưu thay vì yêu cầu nguồn phát lại thủ công. So với chỉ ghi log sự kiện, archive giữ một luồng có thể phát lại qua event bus.
- **Dùng resource-based policy có phạm vi hẹp cho event bus đa tài khoản** → Quyền trên bus quyết định tài khoản/Region nào có thể gửi sự kiện. So với mở quyền rộng, policy giới hạn nguồn giúp giảm nguy cơ sự kiện không mong muốn.

#### KHÔNG NÊN làm (DON'T)

- **Dùng CloudWatch Alarm để bắt mọi sự kiện thay đổi tài nguyên** → Alarm đánh giá metrics, không thay thế rule khớp sự kiện API. NÊN: dùng EventBridge event pattern cho sự kiện như trạng thái EC2 thay đổi; dùng alarm cho ngưỡng CPU/metric.
- **Cho phép event bus truy cập liên tài khoản mà không giới hạn principal** → Tài khoản ngoài phạm vi có thể gửi sự kiện vào bus và kích hoạt target. NÊN: giới hạn principal, nguồn và điều kiện trong resource-based policy.

---

### 5. X-Ray: tracing, sampling và dữ liệu tìm kiếm

#### Giải thích cốt lõi

X-Ray SDK instrument ứng dụng và tạo segments/subsegments; các phần này được ghép thành trace cho yêu cầu end-to-end. Daemon hoặc tích hợp dịch vụ chuyển dữ liệu đến X-Ray để dựng service map. Sampling giảm số traces được ghi nhằm cân bằng độ quan sát với chi phí. Annotations được lập chỉ mục để lọc traces; metadata bổ sung thông tin nhưng không thể tìm kiếm theo cách đó.

CloudWatch metrics cho thấy triệu chứng tổng thể, còn X-Ray thể hiện dịch vụ nào trong đường đi gây độ trễ hoặc lỗi. AWS Distro for OpenTelemetry là lựa chọn khi muốn chuẩn hóa theo OpenTelemetry hoặc gửi telemetry đến nhiều hệ thống.

#### NÊN làm (DO)

- **Bật instrumentation ở các thành phần cần theo dõi và kiểm tra quyền ghi X-Ray** → SDK, daemon/tích hợp và IAM đều cần hoạt động để traces xuất hiện. So với chỉ bật Active Tracing ở Lambda, các thành phần ứng dụng khác trong cùng đường đi cũng phải được instrument nếu muốn trace đầy đủ.
- **Dùng annotations cho thuộc tính cần lọc, metadata cho thông tin bổ sung** → Annotations được lập chỉ mục, metadata thì không. So với lưu mọi dữ liệu vào metadata, annotations giúp tìm trace theo thuộc tính như loại giao dịch.
- **Điều chỉnh sampling theo tải và mục tiêu chẩn đoán** → Sampling mặc định giữ một request mỗi giây cùng một tỷ lệ request bổ sung; rules có thể đổi mà không sửa mã. So với ghi mọi request ở tải cao, sampling giảm khối lượng dữ liệu và chi phí.
- **Dùng OpenTelemetry khi cần giao thức mở hoặc nhiều đích telemetry** → ADOT hỗ trợ APIs, collector và gửi đến nhiều dịch vụ. So với chỉ gắn chặt vào một đích, cách này thuận tiện khi cần chuẩn hóa hoặc chuyển đích.

#### KHÔNG NÊN làm (DON'T)

- **Giả định X-Ray tự ghi trace chỉ vì đã bật ở một dịch vụ** → Thành phần chưa instrument hoặc thiếu quyền có thể không xuất hiện đầy đủ trên service map. NÊN: kiểm tra SDK, daemon/tích hợp và IAM permissions trên từng thành phần.
- **Dùng metadata cho khóa cần truy vấn hoặc annotations cho payload lớn không cần tìm kiếm** → Metadata không được lập chỉ mục; annotations được lập chỉ mục nên cần chọn trường có chủ đích. NÊN: dùng annotation cho khóa lọc và metadata cho chi tiết bổ sung.
- **Tắt sampling mà không cân nhắc lưu lượng và nhu cầu kiểm toán hiệu năng** → Có thể mất mẫu đại diện của đường đi lỗi hoặc bottleneck. NÊN: thiết kế sampling rule theo tải và giữ tỷ lệ phù hợp cho dịch vụ quan trọng.

---

### 6. CloudTrail: kiểm toán hoạt động tài khoản

#### Giải thích cốt lõi

CloudTrail ghi lại hoạt động API trong tài khoản. Management Events bao gồm thao tác quản lý tài nguyên và được trail ghi mặc định; Data Events như truy cập đối tượng S3 hoặc gọi Lambda không được ghi mặc định vì có thể có khối lượng lớn. CloudTrail Insights phân tích baseline hoạt động để nhận diện mẫu bất thường. Event history giữ dữ liệu 90 ngày; cần lưu lâu hơn thì ghi trail ra S3 và dùng Athena để truy vấn.

CloudTrail trả lời “ai gọi API nào và thay đổi gì”; CloudWatch trả lời “metric/log nào đang bất thường”; X-Ray trả lời “yêu cầu bị chậm hoặc lỗi ở bước nào”. Đây là các góc nhìn khác nhau trong cùng một quá trình điều tra.

#### NÊN làm (DO)

- **Bật Data Events cho bucket hoặc Lambda cần kiểm toán ở mức thao tác dữ liệu** → Nếu không bật, hoạt động như `GetObject`, `PutObject`, `DeleteObject` hoặc Lambda `Invoke` không xuất hiện trong nhóm events mặc định. So với chỉ ghi Management Events, bật đúng nguồn Data Events cung cấp chi tiết truy cập dữ liệu cần thiết.
- **Ghi trail vào S3 khi cần lưu lâu hơn 90 ngày** → Event history không giữ dữ liệu lâu dài. So với chỉ dựa vào lịch sử trong console, S3 cùng Athena hỗ trợ lưu trữ và truy vấn dài hạn.
- **Dùng Insights như tín hiệu phát hiện bất thường, sau đó kiểm tra các events gốc** → Insights chỉ chỉ ra mẫu lệch baseline; CloudTrail events cung cấp hoạt động cần điều tra. So với coi anomaly là bằng chứng đầy đủ, xác minh events giúp xác định hành động và principal liên quan.

#### KHÔNG NÊN làm (DON'T)

- **Giả định CloudTrail ghi mọi lần đọc/ghi object S3 theo mặc định** → Data Events mặc định không được ghi, khiến thiếu bằng chứng khi điều tra truy cập object. NÊN: bật đúng Data Events và chọn phạm vi cần kiểm toán.
- **Chỉ dùng CloudTrail Event History để đáp ứng lưu giữ dài hạn** → Lịch sử chỉ lưu 90 ngày theo slide; dữ liệu cũ sẽ không còn trong cửa sổ đó. NÊN: cấu hình trail ghi vào S3 và truy vấn bằng Athena khi cần.
- **Dùng CloudTrail thay cho logs ứng dụng khi tìm stack trace hoặc lỗi nghiệp vụ** → CloudTrail tập trung vào API AWS chứ không phải nội dung xử lý trong code. NÊN: tra CloudWatch Logs/X-Ray cho lỗi ứng dụng, rồi đối chiếu CloudTrail nếu nghi ngờ thay đổi tài nguyên hoặc quyền.
