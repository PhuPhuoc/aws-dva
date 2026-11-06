# Câu hỏi ôn tập DVA-C02 - Section 10: AWS Monitoring, Troubleshooting & Audit

> *Câu hỏi và lựa chọn bằng tiếng Anh; phần giải thích bằng tiếng Việt.*

---

## 1. Thu thập memory metric trên EC2

### Question 1:
An application team needs to send **EC2 memory utilization** to CloudWatch and create an alarm when memory usage is high. What should they do?

```
A. Enable EC2 detailed monitoring
B. Install and configure the CloudWatch Agent to publish custom metrics
C. Create a CloudWatch Logs subscription filter
D. Enable CloudTrail Insights
```

### ✅ Answer: **B. Install and configure the CloudWatch Agent to publish custom metrics**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- EC2 standard metrics không cung cấp memory utilization theo slide.
- CloudWatch Agent thu thập memory từ hệ điều hành và gửi custom metrics lên CloudWatch; alarm sau đó có thể theo dõi metric này.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Detailed monitoring tăng tần suất gửi EC2 metrics, không bổ sung memory metric. |
| C | Subscription filter chuyển log events, không thu thập memory. |
| D | CloudTrail Insights tìm hoạt động API bất thường, không giám sát tài nguyên hệ điều hành. |

### 📝 Vocabulary Analysis:
- **memory utilization**: tỷ lệ dung lượng RAM đang được sử dụng.
- **publish custom metrics**: gửi các chỉ số tự định nghĩa đến CloudWatch.

### 🔄 Context Variations:
- **If the team only needs EC2 CPU metrics at one-minute intervals** → Enable detailed monitoring.
- **If the requirement is to find `ERROR` occurrences in existing logs** → Use CloudWatch Logs Insights for an investigation query.

---

## 2. Tăng tần suất metric EC2 cho Auto Scaling

### Question 2:
An Auto Scaling Group must react to EC2 metric changes faster than the default **five-minute** monitoring interval. Which change provides one-minute EC2 metric reporting?

```
A. Enable detailed monitoring
B. Install the X-Ray daemon
C. Create a CloudTrail trail
D. Configure a CloudWatch Logs metric filter
```

### ✅ Answer: **A. Enable detailed monitoring**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- EC2 gửi standard metrics mỗi 5 phút theo mặc định.
- Detailed monitoring tăng tần suất gửi lên 1 phút, giúp Auto Scaling phản ứng nhanh hơn.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | X-Ray theo dõi yêu cầu ứng dụng, không thay đổi tần suất EC2 metrics. |
| C | CloudTrail ghi hoạt động API và phục vụ kiểm toán. |
| D | Metric filter tạo metric từ log events; không bật chế độ detailed monitoring của EC2. |

### 📝 Vocabulary Analysis:
- **monitoring interval**: khoảng thời gian giữa các lần gửi/cập nhật metric.
- **react faster**: phản ứng nhanh hơn với thay đổi tải.

### 🔄 Context Variations:
- **If the metric is memory utilization** → Install the CloudWatch Agent; detailed monitoring alone does not provide memory data.
- **If one-minute metrics are sufficient** → Use detailed monitoring instead of high-resolution custom metrics.

---

## 3. Tạo alarm từ lỗi trong CloudWatch Logs

### Question 3:
A team must count new application log entries containing **ERROR** and trigger a CloudWatch alarm when the count is too high. Which solution should it use?

```
A. Create a CloudWatch Logs metric filter and alarm on the resulting metric
B. Use Logs Insights to retroactively create an alarm from old log events
C. Enable EC2 detailed monitoring
D. Create an X-Ray sampling rule
```

### ✅ Answer: **A. Create a CloudWatch Logs metric filter and alarm on the resulting metric**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Metric filter đếm log events khớp biểu thức, tạo metric datapoints mới và có thể dùng metric đó làm đầu vào cho CloudWatch Alarm.
- Filter chỉ ghi nhận sự kiện phát sinh sau khi được tạo, đúng với yêu cầu đếm lỗi mới.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | Logs Insights dùng truy vấn dữ liệu; metric filter không hồi tố và Insights không tạo alarm từ dữ liệu cũ. |
| C | Detailed monitoring tăng tần suất EC2 metrics, không đếm nội dung logs. |
| D | Sampling rule kiểm soát số traces gửi đến X-Ray, không lọc application logs. |

### 📝 Vocabulary Analysis:
- **log entry**: một sự kiện/dòng nhật ký.
- **metric filter**: bộ lọc logs tạo metric từ các sự kiện phù hợp.
- **resulting metric**: metric được tạo ra từ bộ lọc.

### 🔄 Context Variations:
- **If the team needs to investigate ERROR entries already stored** → Query them with CloudWatch Logs Insights.
- **If it needs near-real-time delivery of matching logs to another service** → Use a CloudWatch Logs subscription filter.

---

## 4. Chuyển logs gần thời gian thực

### Question 4:
A security team needs to deliver new CloudWatch Logs events to a processing pipeline **near real time**. An S3 export task may take several hours to prepare. What should the team configure?

```
A. A CloudWatch Logs subscription filter to Kinesis Data Firehose
B. A CloudWatch Logs S3 export task
C. A CloudWatch Logs retention policy of 10 years
D. A CloudTrail management event selector
```

### ✅ Answer: **A. A CloudWatch Logs subscription filter to Kinesis Data Firehose**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Logs Subscriptions chuyển log events gần thời gian thực đến Kinesis Data Firehose, Kinesis Data Streams hoặc Lambda.
- Subscription Filter có thể chọn những log events cần gửi.
- S3 export task có thể cần đến 12 giờ để dữ liệu sẵn sàng nên không phù hợp với pipeline gần thời gian thực.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | S3 export task là xuất dữ liệu theo tác vụ, có thể trễ đến 12 giờ. |
| C | Retention policy quyết định thời gian lưu, không chuyển log events đến pipeline. |
| D | CloudTrail management event selector theo dõi hoạt động quản lý AWS, không chuyển CloudWatch Logs. |

### 📝 Vocabulary Analysis:
- **near real time**: gần thời gian thực, có độ trễ ngắn nhưng không tức thời tuyệt đối.
- **subscription filter**: bộ lọc chọn log events để chuyển đến đích đăng ký.
- **export task**: tác vụ xuất dữ liệu logs sang S3.

### 🔄 Context Variations:
- **If logs only need to be queried interactively** → Use CloudWatch Logs Insights.
- **If logs need long-term storage rather than continuous processing** → Export/store them in S3 with an appropriate retention plan.

---

## 5. Phát hiện thay đổi trạng thái EC2 bằng EventBridge

### Question 5:
An operations team must notify an SNS topic whenever an EC2 instance changes state. Which configuration is the best fit?

```
A. A CloudWatch alarm on CPU utilization
B. An EventBridge rule with an EC2 state-change event pattern and an SNS target
C. An X-Ray sampling rule
D. A CloudTrail Insights event selector
```

### ✅ Answer: **B. An EventBridge rule with an EC2 state-change event pattern and an SNS target**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- EventBridge Event Pattern khớp sự kiện thay đổi trạng thái EC2; rule có thể gửi sự kiện đến SNS.
- Đây là phản ứng theo sự kiện, không cần đợi metric vượt ngưỡng.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | CloudWatch Alarm theo dõi metric như CPU, không trực tiếp khớp mọi sự kiện đổi trạng thái. |
| C | X-Ray sampling rule giới hạn traces được ghi lại. |
| D | CloudTrail Insights phát hiện hoạt động API bất thường, không thay thế EventBridge rule định tuyến sự kiện. |

### 📝 Vocabulary Analysis:
- **state change**: thay đổi trạng thái của tài nguyên.
- **event pattern**: điều kiện khớp cấu trúc/nội dung sự kiện.
- **target**: đích nhận sự kiện từ EventBridge rule.

### 🔄 Context Variations:
- **If the team needs to notify on CPU remaining above a threshold for a period** → Use a CloudWatch Alarm.
- **If the team needs a scheduled task rather than a response to an event** → Configure an EventBridge schedule.

---

## 6. Lưu trữ và phát lại sự kiện EventBridge

### Question 6:
A team needs to retain selected events from an EventBridge event bus and **replay** them after fixing a downstream processing bug. Which feature should it use?

```
A. EventBridge archive and replay
B. CloudWatch detailed monitoring
C. CloudTrail Event History
D. X-Ray annotations
```

### ✅ Answer: **A. EventBridge archive and replay**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- EventBridge có thể lưu toàn bộ hoặc các sự kiện khớp bộ lọc trong khoảng thời gian đã chọn hoặc vô thời hạn.
- Có thể phát lại các sự kiện đã lưu sau khi sửa hệ thống xử lý.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | Detailed monitoring tăng tần suất metrics EC2, không lưu sự kiện EventBridge. |
| C | CloudTrail Event History ghi hoạt động API để kiểm toán, không phải archive sự kiện bus để replay. |
| D | Annotation bổ sung thuộc tính tìm traces trong X-Ray, không lưu sự kiện ứng dụng. |

### 📝 Vocabulary Analysis:
- **archive**: lưu trữ sự kiện để dùng lại.
- **replay**: phát lại sự kiện đã lưu vào luồng xử lý.
- **downstream**: thành phần xử lý ở phía sau trong pipeline.

### 🔄 Context Variations:
- **If the requirement is durable work distribution to independent consumers** → Use SQS queues.
- **If a rule should match future events but no replay is needed** → Configure an EventBridge event pattern without an archive.

---

## 7. Tìm điểm nghẽn trong ứng dụng phân tán

### Question 7:
A request passes through an API, several microservices, and a database. The team needs to identify **which service adds latency** to the request. Which service should it use?

```
A. AWS CloudTrail
B. Amazon CloudWatch Synthetics only
C. AWS X-Ray with application instrumentation
D. CloudWatch EC2 detailed monitoring
```

### ✅ Answer: **C. AWS X-Ray with application instrumentation**

### 📖 Explanation (Vietnamese):

**Tại sao chọn C:**
- X-Ray ghép các segments thành trace end-to-end và hiển thị service map, giúp xác định thành phần hoặc lời gọi gây độ trễ.
- Cần instrument ứng dụng bằng X-Ray SDK hoặc bật tích hợp phù hợp; các thành phần cần quyền gửi dữ liệu đến X-Ray.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | CloudTrail kiểm toán API AWS và thay đổi tài nguyên, không đo thời gian từng bước của request ứng dụng. |
| B | Synthetics Canary mô phỏng hành vi người dùng để kiểm tra endpoint, không cung cấp phân tích đầy đủ các service nội bộ như trace đã instrument. |
| D | Detailed monitoring cung cấp EC2 metrics theo phút, không theo dấu request qua nhiều dịch vụ. |

### 📝 Vocabulary Analysis:
- **latency**: độ trễ của thao tác hoặc yêu cầu.
- **instrumentation**: bổ sung SDK/cấu hình để ứng dụng phát dữ liệu quan sát.
- **trace**: tập hợp dữ liệu theo dõi một request từ đầu đến cuối.

### 🔄 Context Variations:
- **If only endpoint availability and response time need periodic checks** → Use CloudWatch Synthetics Canary.
- **If the team needs aggregate CPU trends** → Use CloudWatch Metrics and Alarms.

---

## 8. Chọn Annotation hay Metadata trong X-Ray

### Question 8:
A team needs to filter X-Ray traces by a **customer tier** value. Which trace field should it add?

```
A. Metadata
B. Annotation
C. Log stream
D. CloudTrail data event
```

### ✅ Answer: **B. Annotation**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- X-Ray annotations là các cặp khóa-giá trị được lập chỉ mục, có thể dùng để lọc traces.
- Metadata bổ sung dữ liệu nhưng không được lập chỉ mục để tìm kiếm.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Metadata không được lập chỉ mục, nên không dùng làm thuộc tính lọc trace. |
| C | Log stream chứa chuỗi sự kiện nhật ký CloudWatch Logs, không phải trường tìm kiếm của trace. |
| D | Data event ghi hoạt động tài nguyên như S3/Lambda trong CloudTrail, không thêm thuộc tính vào X-Ray trace. |

### 📝 Vocabulary Analysis:
- **annotation**: thuộc tính trace được lập chỉ mục để tìm kiếm/lọc.
- **metadata**: thông tin bổ sung của trace không được lập chỉ mục.
- **customer tier**: cấp/nhóm khách hàng, ví dụ tiêu chuẩn hoặc cao cấp.

### 🔄 Context Variations:
- **If the value is extra diagnostic context that does not need filtering** → Add it as metadata.
- **If trace volume must be reduced** → Adjust X-Ray sampling rules.

---

## 9. Kiểm toán thao tác trên đối tượng S3

### Question 9:
A security team must identify who called **DeleteObject** on an S3 object and retain the audit records for one year. The trail currently records management events. What should the team do?

```
A. Enable S3 object-level data events and deliver the trail to S3 for long-term retention
B. Enable EC2 detailed monitoring and keep metrics for one year
C. Create a CloudWatch Logs metric filter for CPU utilization
D. Enable X-Ray Active Tracing on the S3 bucket
```

### ✅ Answer: **A. Enable S3 object-level data events and deliver the trail to S3 for long-term retention**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- `DeleteObject` là thao tác ở mức đối tượng S3, thuộc CloudTrail Data Events; Data Events không được ghi mặc định.
- CloudTrail Event History lưu events 90 ngày theo slide. Ghi trail vào S3 để giữ lâu hơn; có thể dùng Athena để truy vấn.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | Detailed monitoring thu thập EC2 metrics, không ghi danh tính gọi S3 `DeleteObject`. |
| C | Metric filter tạo metric từ logs, không bật CloudTrail Data Events. |
| D | X-Ray theo dõi request ứng dụng, không kiểm toán thao tác đối tượng S3. |

### 📝 Vocabulary Analysis:
- **object-level data event**: sự kiện truy cập/thao tác trên đối tượng S3.
- **audit record**: bản ghi phục vụ kiểm toán.
- **retain**: lưu giữ dữ liệu trong thời gian yêu cầu.

### 🔄 Context Variations:
- **If the investigation concerns who created an IAM role or changed a security setting** → Review CloudTrail management events.
- **If the team needs to query records retained in S3** → Use Athena against the stored CloudTrail logs.

---

## 10. Phát hiện hoạt động API bất thường

### Question 10:
An account experiences an unusual burst of IAM API actions compared with its normal activity. Which CloudTrail capability can detect this anomaly?

```
A. CloudTrail Insights
B. CloudWatch Logs Insights
C. X-Ray service map
D. EventBridge Schema Registry
```

### ✅ Answer: **A. CloudTrail Insights**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- CloudTrail Insights tạo baseline từ management events bình thường rồi phân tích write events để phát hiện hoạt động bất thường, bao gồm đột biến thao tác IAM.
- Anomaly có thể xuất hiện trong CloudTrail console, được gửi đến S3 và tạo EventBridge event.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | Logs Insights truy vấn CloudWatch Logs; không phân tích baseline hoạt động API của tài khoản. |
| C | X-Ray service map biểu diễn đường đi request và phụ thuộc dịch vụ, không phát hiện bất thường IAM. |
| D | Schema Registry suy ra cấu trúc sự kiện và hỗ trợ tạo mã; không phát hiện hoạt động bất thường. |

### 📝 Vocabulary Analysis:
- **burst**: đợt tăng đột biến trong thời gian ngắn.
- **baseline**: mức hoạt động bình thường dùng làm cơ sở so sánh.
- **anomaly**: hoạt động sai khác đáng kể so với mẫu thông thường.

### 🔄 Context Variations:
- **If a team needs the identity and API call that changed a resource** → Review the relevant CloudTrail event.
- **If a team needs an alert when a metric crosses a threshold** → Use a CloudWatch Alarm.
