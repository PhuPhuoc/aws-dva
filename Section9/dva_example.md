# Câu hỏi ôn tập DVA-C02 - Section 9: AWS Integration & Messaging

> *Câu hỏi và lựa chọn bằng tiếng Anh; phần giải thích bằng tiếng Việt.*

---

## 1. Tách rời công việc bằng SQS

### Question 1:
A video-processing application normally receives 10 jobs per hour but can suddenly receive **1,000 jobs**. The processing workers should scale independently from the API application. Which service should buffer the jobs?

```
A. Amazon SQS
B. Amazon SNS
C. Amazon Data Firehose
D. AWS CloudTrail
```

### ✅ Answer: **A. Amazon SQS**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- SQS cung cấp mô hình hàng đợi để tách bên gửi công việc khỏi bên xử lý.
- API có thể gửi tác vụ vào hàng đợi; các máy xử lý nhận dần và có thể mở rộng riêng theo lượng công việc.
- Đây là lựa chọn phù hợp cho từng tác vụ cần được xử lý, không phải phân phối cùng một sự kiện cho nhiều bên nhận.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B. Amazon SNS | Mô hình xuất bản/đăng ký nhận phù hợp để gửi một thông báo đến nhiều bên nhận; không phải hàng đợi công việc độc lập cho worker. |
| C. Amazon Data Firehose | Dùng để chuyển dữ liệu liên tục đến đích; không phải hàng đợi tác vụ cho bên xử lý. |
| D. AWS CloudTrail | Ghi nhận hoạt động API và kiểm toán AWS, không đệm tác vụ ứng dụng. |

### 📝 Vocabulary Analysis:
- **buffer**: lưu tạm công việc để xử lý sau.
- **scale independently**: mở rộng riêng từng thành phần mà không phải mở rộng đồng thời các thành phần khác.
- **processing worker**: tiến trình hoặc instance nhận và xử lý công việc.

### 🔄 Context Variations:
- **If every downstream system must receive the same event** → Use an SNS topic, optionally with one SQS queue per subscriber.
- **If consumers must replay retained streaming records** → Use Kinesis Data Streams.

---

## 2. SQS Visibility Timeout và DLQ

### Question 2:
An SQS consumer receives a message but does not delete it before the **visibility timeout** expires. What happens next?

```
A. The message is permanently deleted.
B. The message becomes visible in the queue again.
C. The message is automatically sent to a DLQ after the first timeout.
D. The queue changes to FIFO mode.
```

### ✅ Answer: **B. The message becomes visible in the queue again.**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Visibility Timeout chỉ làm thông điệp tạm thời ẩn sau khi bên xử lý nhận nó.
- Nếu bên xử lý không xóa thông điệp trước khi hết thời gian, thông điệp xuất hiện lại để có thể được xử lý lần nữa.
- Thông điệp chỉ được chuyển sang DLQ sau khi vượt ngưỡng số lần nhận đã cấu hình, không phải tự động sau lần hết hạn đầu tiên.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Thông điệp chỉ bị xóa khi bên xử lý gọi `DeleteMessage`. |
| C | DLQ phụ thuộc vào redrive policy và ngưỡng số lần nhận. |
| D | Visibility Timeout không thay đổi loại hàng đợi. |

### 📝 Vocabulary Analysis:
- **visibility timeout**: khoảng thời gian thông điệp bị ẩn sau khi bên xử lý nhận.
- **expire**: hết hạn.
- **redrive policy**: cấu hình chuyển thông điệp từ hàng đợi nguồn sang DLQ sau số lần nhận tối đa.

### 🔄 Context Variations:
- **If the message has exceeded the configured maximum receive count** → The redrive policy can move it to the DLQ.
- **If processing needs more time but is still healthy** → The consumer can call `ChangeMessageVisibility`.

---

## 3. SQS Long Polling

### Question 3:
An application makes many empty `ReceiveMessage` calls when its SQS queue has no messages. Which change reduces these empty responses and API calls?

```
A. Set `WaitTimeSeconds` to enable long polling.
B. Set `DelaySeconds` on every message.
C. Call `ChangeMessageVisibility` more frequently.
D. Configure a dead-letter queue.
```

### ✅ Answer: **A. Set `WaitTimeSeconds` to enable long polling.**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Long Polling cho phép `ReceiveMessage` chờ thông điệp đến thay vì trả kết quả rỗng ngay.
- `WaitTimeSeconds` có thể đặt thời gian chờ từ 1 đến 20 giây; slide khuyến nghị 20 giây.
- Nhờ đó bên xử lý giảm số lần gọi API khi hàng đợi rỗng.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | `DelaySeconds` trì hoãn lúc thông điệp xuất hiện, không giảm các lần thăm dò hàng đợi rỗng. |
| C | `ChangeMessageVisibility` thay đổi thời gian ẩn của thông điệp đã nhận. |
| D | DLQ xử lý thông điệp lỗi sau nhiều lần nhận, không thay đổi cách thăm dò. |

### 📝 Vocabulary Analysis:
- **empty response**: phản hồi không có thông điệp.
- **long polling**: giữ yêu cầu nhận thông điệp trong một khoảng chờ để giảm phản hồi rỗng.
- **frequently**: thường xuyên, với tần suất cao.

### 🔄 Context Variations:
- **If a message must not be visible immediately after it is sent** → Set a queue-level delay or `DelaySeconds`.
- **If a received message needs more processing time** → Change its visibility timeout.

---

## 4. Nhóm thông điệp trong SQS FIFO

### Question 4:
An order-processing system must preserve message order **per customer**, while allowing orders from different customers to be processed in parallel. How should the application configure an SQS FIFO queue?

```
A. Use one `MessageGroupId` for all customers.
B. Use each customer's identifier as the `MessageGroupId`.
C. Use a different queue for every message.
D. Use a Standard queue and set `DelaySeconds`.
```

### ✅ Answer: **B. Use each customer's identifier as the `MessageGroupId`.**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Trong hàng đợi FIFO, thông điệp có cùng `MessageGroupId` được xử lý theo thứ tự trong nhóm.
- Dùng mã khách hàng làm mã nhóm giữ thứ tự của từng khách hàng, đồng thời cho phép các nhóm khách hàng khác nhau xử lý song song.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Một nhóm cho mọi khách hàng tuần tự hóa toàn bộ hàng đợi, làm mất khả năng xử lý song song giữa khách hàng. |
| C | Một hàng đợi cho từng thông điệp không cần thiết và không dùng cơ chế nhóm của FIFO. |
| D | Hàng đợi tiêu chuẩn không đảm bảo thứ tự; trì hoãn chỉ thay đổi thời điểm thông điệp xuất hiện. |

### 📝 Vocabulary Analysis:
- **preserve order**: giữ nguyên thứ tự xử lý.
- **per customer**: riêng cho từng khách hàng.
- **in parallel**: thực hiện song song.

### 🔄 Context Variations:
- **If strict ordering is not required and throughput is the priority** → Consider an SQS Standard queue.
- **If all messages must be ordered globally** → A single FIFO message group preserves that order but limits parallel processing.

---

## 5. SNS phân phối đến nhiều bên nhận độc lập

### Question 5:
An S3 event with the same event type and prefix must be delivered to several independent processing systems. Each system needs its own durable queue. Which design best meets the requirement?

```
A. Configure several identical S3 event rules that each target a different queue.
B. Send the event to one SNS topic and subscribe one SQS queue per system.
C. Send the event to one SQS queue and have all systems delete messages from it.
D. Store the event only in CloudTrail.
```

### ✅ Answer: **B. Send the event to one SNS topic and subscribe one SQS queue per system.**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Mô hình SNS phân phối một-nhiều gửi một lần đến nhiều bên đăng ký nhận.
- Mỗi hệ thống có SQS queue riêng để xử lý thông điệp độc lập, lưu thông điệp và xử lý trễ.
- Slide nêu cùng một tổ hợp loại sự kiện S3 và prefix chỉ có thể có một event rule; SNS phân phối một-nhiều là cách gửi cùng sự kiện đến nhiều hàng đợi.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Không phù hợp với giới hạn một rule cho cùng loại sự kiện và prefix. |
| C | Các bên xử lý cùng thăm dò một hàng đợi sẽ cạnh tranh nhận thông điệp thay vì mỗi hệ thống nhận bản riêng. |
| D | CloudTrail kiểm toán hoạt động API, không phân phối sự kiện S3 đến hàng đợi ứng dụng. |

### 📝 Vocabulary Analysis:
- **fan-out**: phân phối một thông điệp hoặc sự kiện đến nhiều nhánh nhận.
- **durable queue**: hàng đợi lưu thông điệp để bên xử lý nhận sau.
- **subscribe**: đăng ký nhận thông điệp từ SNS topic.

### 🔄 Context Variations:
- **If only one consumer needs the task** → Send the task to one SQS queue directly.
- **If subscribers should receive only selected events** → Add an SNS message filter policy to each subscription.

---

## 6. Lọc thông điệp SNS

### Question 6:
An SNS topic publishes messages for several product categories. A subscriber should receive only messages whose JSON attributes match its category. Which feature should be configured?

```
A. SQS Visibility Timeout
B. SNS message filter policy
C. SNS FIFO MessageGroupId
D. Kinesis shard retention
```

### ✅ Answer: **B. SNS message filter policy**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Chính sách lọc của SNS lọc thông điệp theo tiêu chí JSON ở lượt đăng ký nhận.
- Bên nhận có bộ lọc chỉ nhận thông điệp phù hợp; lượt đăng ký không có bộ lọc nhận mọi thông điệp của topic.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Visibility Timeout thuộc SQS và điều khiển thời gian thông điệp bị ẩn sau khi nhận. |
| C | `MessageGroupId` liên quan đến thứ tự của SNS FIFO, không dùng để lọc bên nhận. |
| D | Thời gian lưu của shard thuộc Kinesis Data Streams, không lọc thông điệp SNS. |

### 📝 Vocabulary Analysis:
- **attribute**: thuộc tính được gắn với thông điệp.
- **match**: khớp với điều kiện lọc.
- **filter policy**: chính sách xác định thông điệp nào được chuyển đến lượt đăng ký nhận.

### 🔄 Context Variations:
- **If a subscriber must receive every topic message** → Do not configure a filter policy for that subscription.
- **If ordering and deduplication are also required** → Consider an SNS FIFO topic, while retaining a filter policy if selective delivery is needed.

---

## 7. Phát lại dữ liệu luồng

### Question 7:
A real-time analytics application must retain streaming records and allow a consumer to **replay** records for up to a year. Which service should be used?

```
A. Amazon SQS Standard
B. Amazon SNS
C. Kinesis Data Streams
D. Amazon Data Firehose
```

### ✅ Answer: **C. Kinesis Data Streams**

### 📖 Explanation (Vietnamese):

**Tại sao chọn C:**
- Kinesis Data Streams lưu bản ghi trong khoảng thời gian từ 1 đến 365 ngày theo slide.
- Bên xử lý có thể đọc lại dữ liệu còn trong thời gian lưu; thứ tự được duy trì cho dữ liệu cùng Partition ID.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | SQS Standard là hàng đợi công việc, không cung cấp khả năng phát lại dữ liệu luồng đến một năm. |
| B | SNS phân phối thông điệp đến bên nhận nhưng không phải kho dữ liệu luồng có thời gian lưu và khả năng phát lại. |
| D | Data Firehose chuyển dữ liệu đến đích nhưng slide nêu không lưu luồng và không hỗ trợ phát lại. |

### 📝 Vocabulary Analysis:
- **streaming record**: bản ghi dữ liệu được tạo liên tục theo dòng sự kiện.
- **retain**: lưu giữ trong một khoảng thời gian.
- **replay**: đọc hoặc phát lại dữ liệu đã lưu.
- **Partition ID**: khóa phân vùng dùng để xác định thứ tự và phân phối dữ liệu.

### 🔄 Context Variations:
- **If the requirement is managed delivery to S3 or Redshift without replay** → Use Amazon Data Firehose.
- **If only individual background jobs need retry and deletion after processing** → Use SQS.

---

## 8. Chuyển dữ liệu luồng được quản lý

### Question 8:
A team needs to deliver streaming data to Amazon S3 with managed buffering and optional format conversion. It does not need to replay the stream. Which service is the best fit?

```
A. Kinesis Data Streams
B. Amazon Data Firehose
C. Amazon SNS FIFO
D. SQS Extended Client
```

### ✅ Answer: **B. Amazon Data Firehose**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Amazon Data Firehose là dịch vụ được quản lý để chuyển dữ liệu liên tục đến các đích như S3.
- Hỗ trợ đệm dữ liệu, giao gần thời gian thực, chuyển đổi định dạng và biến đổi tùy chỉnh bằng Lambda.
- Đề bài không yêu cầu phát lại nên không cần chọn Data Streams chỉ để giữ dữ liệu có thể đọc lại.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Data Streams phù hợp khi cần tự quản lý bên xử lý hoặc phát lại; đề bài không yêu cầu phát lại. |
| C | SNS FIFO phân phối thông điệp theo mô hình xuất bản/đăng ký nhận có thứ tự, không phải dịch vụ chuyển dữ liệu đến S3. |
| D | SQS Extended Client hỗ trợ phần dữ liệu lớn qua S3 cho SQS; không phải đường ống chuyển dữ liệu liên tục. |

### 📝 Vocabulary Analysis:
- **buffering**: gom dữ liệu tạm thời trước khi chuyển đến đích.
- **format conversion**: chuyển đổi định dạng dữ liệu.
- **delivery**: chuyển dữ liệu đến hệ thống đích.

### 🔄 Context Variations:
- **If consumers need to read and replay records from retained shards** → Use Kinesis Data Streams.
- **If records need continuous stateful stream processing with Apache Flink** → Use Amazon Managed Service for Apache Flink with a supported streaming source.

---

## 9. Apache Flink được quản lý

### Question 9:
A team needs to run a Java application that continuously processes streaming data using Apache Flink, with managed checkpoints and snapshots. Which AWS service should it use?

```
A. Amazon Managed Service for Apache Flink
B. Amazon Data Firehose
C. Amazon SQS
D. Amazon SNS
```

### ✅ Answer: **A. Amazon Managed Service for Apache Flink**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Dịch vụ quản lý ứng dụng Apache Flink trên AWS, hỗ trợ xử lý song song và điểm kiểm tra/bản chụp trạng thái.
- Slide nêu Flink không đọc trực tiếp từ Amazon Data Firehose; cần thiết kế đầu vào từ một nguồn dữ liệu liên tục được hỗ trợ.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | Data Firehose chuyển dữ liệu đến đích; không chạy ứng dụng Apache Flink. |
| C | SQS là hàng đợi công việc, không phải môi trường chạy Flink để xử lý dữ liệu liên tục. |
| D | SNS là dịch vụ thông báo theo mô hình xuất bản/đăng ký nhận, không chạy mã xử lý luồng liên tục. |

### 📝 Vocabulary Analysis:
- **continuously processes**: xử lý liên tục khi dữ liệu mới đến.
- **checkpoint**: điểm lưu trạng thái để phục hồi tiến trình xử lý.
- **snapshot**: bản chụp trạng thái ứng dụng tại một thời điểm.

### 🔄 Context Variations:
- **If data only needs to be delivered to a supported destination with minimal pipeline management** → Use Amazon Data Firehose.
- **If the requirement is retaining and replaying raw stream records** → Use Kinesis Data Streams.
