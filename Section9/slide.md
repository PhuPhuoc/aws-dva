# Section 9 - AWS: Tích hợp và trao đổi thông điệp qua SQS, SNS và Kinesis (Trang 435-477)

---

## Giới thiệu: Giao tiếp giữa các ứng dụng (Trang 435-437)

- Khi triển khai nhiều ứng dụng, các ứng dụng cần giao tiếp với nhau.
- Giao tiếp đồng bộ có thể gặp vấn đề khi lưu lượng tăng đột ngột. Ví dụ: hệ thống thường mã hóa 10 video nhưng bất ngờ phải mã hóa 1.000 video.
- Có thể tách rời các ứng dụng để từng dịch vụ mở rộng độc lập:
  - **SQS**: mô hình hàng đợi.
  - **SNS**: mô hình xuất bản/đăng ký nhận (pub/sub).
  - **Kinesis**: mô hình xử lý dữ liệu liên tục theo thời gian thực.

## Phần 1: Amazon SQS (Trang 438-457)

### Hàng đợi tiêu chuẩn (Trang 438-439)

> *Sơ đồ: Bên gửi đưa thông điệp vào hàng đợi; bên xử lý nhận và xử lý thông điệp.*

- SQS là dịch vụ được quản lý hoàn toàn, dùng để tách rời các ứng dụng.
- Hàng đợi tiêu chuẩn có thông lượng và số lượng thông điệp không giới hạn.
- Thời gian lưu thông điệp mặc định là **4 ngày**, tối đa **14 ngày**.
- Độ trễ khi gửi và nhận thấp, dưới **10 ms**.
- Kích thước thông điệp tối đa là **256 KB**.
- Có thể nhận thông điệp trùng lặp vì cơ chế giao nhận **at-least-once**.
- Thứ tự thông điệp chỉ được đảm bảo ở mức tốt nhất có thể (**best effort**).

### Gửi thông điệp vào SQS (Trang 440)

- Bên gửi đưa thông điệp vào SQS bằng **SDK** và API `SendMessage`.
- Thông điệp được lưu trong SQS cho đến khi bên xử lý xóa bằng `DeleteMessage`.
- Thời gian lưu: mặc định **4 ngày**, tối đa **14 ngày**.
- Ví dụ thông điệp đơn hàng có thể gồm mã đơn hàng, mã khách hàng và các thuộc tính cần thiết.
- Hàng đợi tiêu chuẩn hỗ trợ thông lượng không giới hạn.
- Kích thước thông điệp tối đa là **256 KB**.

### Bên xử lý và xử lý song song (Trang 441-444)

- Bên xử lý chạy trên EC2, máy chủ hoặc Lambda có thể:
  1. Thăm dò SQS và nhận tối đa **10 thông điệp** mỗi lần.
  2. Xử lý thông điệp, ví dụ ghi thông tin vào RDS.
  3. Xóa thông điệp bằng API `DeleteMessage` sau khi xử lý.
- Nhiều EC2 instance có thể nhận và xử lý thông điệp song song.
- SQS tiêu chuẩn có cơ chế giao nhận **at-least-once**; thứ tự chỉ được đảm bảo ở mức tốt nhất có thể.
- Có thể mở rộng ngang các bên xử lý để tăng thông lượng.

> *Sơ đồ (Trang 443): Các bên xử lý trong Auto Scaling Group xử lý thông điệp từ SQS.*
>
> *Sơ đồ (Trang 444): SQS tách rời các tầng của ứng dụng.*

### Bảo mật SQS (Trang 445-446)

- **Mã hóa khi truyền:** dùng HTTPS API.
- **Mã hóa khi lưu trữ:** dùng KMS keys.
- **Mã hóa phía client:** client tự mã hóa và giải mã nếu cần.
- **IAM policies** điều chỉnh quyền gọi SQS API.
- **SQS Access Policies** hỗ trợ truy cập giữa các tài khoản và cho phép dịch vụ như SNS hoặc S3 gửi thông điệp vào hàng đợi.

### Thời gian ẩn thông điệp (Visibility Timeout) (Trang 447-448)

- Sau khi bên xử lý nhận thông điệp, thông điệp trở nên ẩn với các bên xử lý khác trong khoảng **Visibility Timeout**.
- Giá trị mặc định là **30 giây**.
- Nếu bên xử lý không xóa thông điệp trước khi hết thời gian, thông điệp hiển thị lại trong hàng đợi và có thể được xử lý lần nữa.
- Bên xử lý có thể gọi `ChangeMessageVisibility` để kéo dài thời gian xử lý.
- Thời gian quá ngắn có thể dẫn đến xử lý trùng lặp; thời gian quá dài làm chậm việc xử lý lại nếu bên xử lý gặp sự cố.

### Hàng đợi thông điệp lỗi (DLQ) và đưa lại thông điệp (Trang 449-450)

- Khi xử lý thất bại, thông điệp trở lại hàng đợi sau khi hết Visibility Timeout.
- Có thể đặt ngưỡng số lần nhận tối đa (**MaximumReceives**). Khi vượt ngưỡng, thông điệp được chuyển sang **Dead Letter Queue (DLQ)** để điều tra lỗi.
- DLQ phải cùng loại với hàng đợi nguồn: hàng đợi tiêu chuẩn dùng DLQ tiêu chuẩn; FIFO dùng FIFO DLQ.
- Nên đặt thời gian lưu DLQ đủ dài; slide khuyến nghị tối đa **14 ngày** để có thời gian xử lý thông điệp lỗi.
- **Redrive to source** đưa thông điệp từ DLQ về hàng đợi nguồn hoặc hàng đợi khác theo lô sau khi đã tìm và khắc phục nguyên nhân.

### Hàng đợi trì hoãn thông điệp (Trang 451)

- Trì hoãn thời điểm bên xử lý nhìn thấy thông điệp tối đa **15 phút**.
- Mặc định là **0 giây**, tức thông điệp có thể được nhận ngay.
- Có thể đặt mặc định ở cấp hàng đợi hoặc ghi đè cho từng thông điệp bằng `DelaySeconds`.

### Thăm dò dài (Long Polling) (Trang 452)

- Khi hàng đợi chưa có thông điệp, bên xử lý có thể chờ thông điệp đến thay vì nhận phản hồi ngay; cách này gọi là **Long Polling**.
- Thời gian chờ từ **1 đến 20 giây**; slide khuyến nghị 20 giây.
- Long Polling giảm số lần gọi API và tăng hiệu quả sử dụng SQS.
- Có thể bật ở cấp hàng đợi hoặc truyền `WaitTimeSeconds` khi gọi API.

### SQS Extended Client (Trang 453)

- Giới hạn kích thước thông điệp SQS là **256 KB**.
- Với phần dữ liệu lớn, ví dụ **1 GB**, thư viện Java **SQS Extended Client** lưu dữ liệu trong S3 và gửi tham chiếu qua SQS.

### Các API SQS cần biết (Trang 454)

| API | Mục đích |
|-----|----------|
| `CreateQueue` | Tạo hàng đợi và cấu hình như `MessageRetentionPeriod` |
| `DeleteQueue` | Xóa hàng đợi |
| `PurgeQueue` | Xóa toàn bộ thông điệp trong hàng đợi |
| `SendMessage` | Gửi thông điệp; có thể đặt `DelaySeconds` |
| `ReceiveMessage` | Nhận thông điệp; tối đa 10 thông điệp mỗi lần |
| `DeleteMessage` | Xóa thông điệp sau khi xử lý |
| `ChangeMessageVisibility` | Thay đổi Visibility Timeout |

- Các API theo lô cho `SendMessage`, `DeleteMessage` và `ChangeMessageVisibility` giúp giảm chi phí gọi API.

### Hàng đợi FIFO (Trang 455-457)

- FIFO là **First In, First Out**: giữ thứ tự thông điệp.
- Thông lượng tối đa theo nội dung slide: **300 thông điệp/giây** không dùng batching, **3.000 thông điệp/giây** khi batching.
- Hỗ trợ loại bỏ thông điệp gửi trùng lặp trong khoảng thời gian khử trùng lặp **5 phút**.
- Hai cách khử trùng lặp:
  - **Dựa trên nội dung:** tạo SHA-256 hash từ nội dung thông điệp.
  - **Do bên gửi cung cấp:** dùng `MessageDeduplicationId`.
- Các thông điệp có cùng `MessageGroupId` được xử lý đúng thứ tự trong cùng nhóm; mỗi nhóm chỉ có một bên xử lý tại một thời điểm.
- Các nhóm khác nhau có thể được xử lý song song; không đảm bảo thứ tự giữa các nhóm.

## Phần 2: Amazon SNS (Trang 458-468)

### Tổng quan SNS và bên đăng ký nhận (Trang 458-459)

- Bên gửi chỉ cần gửi thông điệp đến một SNS topic; nhiều bên đăng ký nhận có thể lắng nghe topic đó.
- Bên đăng ký nhận topic sẽ nhận thông điệp được gửi đến topic, trừ khi có cấu hình lọc.
- Mỗi topic hỗ trợ tối đa **12.500.000 lượt đăng ký**; giới hạn là **100.000 topics** mỗi tài khoản.
- Bên nhận có thể là SQS, Lambda, Kinesis Data Firehose, email, SMS, HTTP(S) hoặc điểm nhận thông báo di động.

### Tích hợp SNS và cách gửi thông điệp (Trang 460-461)

- Các dịch vụ có thể gửi sự kiện trực tiếp đến SNS gồm CloudWatch Alarms, AWS Budgets, Auto Scaling Group, sự kiện S3, thay đổi trạng thái CloudFormation, AWS DMS, Lambda, DynamoDB và RDS.
- **Gửi đến topic:** tạo topic, tạo một hoặc nhiều lượt đăng ký nhận, rồi gửi thông điệp đến topic.
- **Gửi trực tiếp đến ứng dụng di động:** tạo ứng dụng nền tảng, tạo điểm nhận nền tảng, rồi gửi thông điệp đến điểm nhận. Các nền tảng gồm Apple APNS, Google GCM và Amazon ADM.

### Bảo mật SNS (Trang 462)

- **Mã hóa khi truyền:** HTTPS.
- **Mã hóa khi lưu trữ:** KMS keys.
- Có thể dùng mã hóa phía client.
- **IAM policies** kiểm soát quyền gọi SNS API.
- **SNS Access Policies** hỗ trợ truy cập giữa các tài khoản và cho phép dịch vụ như S3 gửi thông điệp đến topic.

### SNS kết hợp SQS: Phân phối một-nhiều (fan-out) (Trang 463-465)

- SNS gửi một thông điệp đến nhiều SQS queues đã đăng ký nhận.
- Mỗi hàng đợi có thể phục vụ một bên xử lý hoặc một luồng xử lý riêng.
- SQS cung cấp lưu trữ thông điệp, xử lý trễ và khả năng thử lại công việc.
- Có thể thêm bên nhận SQS mới; hỗ trợ gửi đến SQS queue ở Region khác.
- Để S3 gửi cùng một loại sự kiện với cùng prefix đến nhiều SQS queues, dùng mô hình phân phối một-nhiều.
- Có thể đưa thông điệp từ SNS qua Kinesis Data Firehose để chuyển dữ liệu đến nơi lưu trữ hoặc phân tích.

> *Sơ đồ: Một SNS topic phân phối cùng thông điệp đến nhiều SQS queues; mỗi hàng đợi phục vụ một bên xử lý riêng.*

### SNS FIFO và lọc thông điệp (Trang 466-468)

- SNS FIFO giữ thứ tự theo `MessageGroupId` và hỗ trợ khử trùng lặp bằng ID hoặc cách dựa trên nội dung.
- Có thể dùng cả SQS tiêu chuẩn và SQS FIFO làm bên nhận; thông lượng giới hạn như SQS FIFO theo nội dung slide.
- Kết hợp SNS FIFO với SQS FIFO khi cần phân phối một-nhiều kèm thứ tự và khử trùng lặp.
- **Message Filtering** dùng JSON filter policy để chỉ chuyển thông điệp phù hợp đến lượt đăng ký.
- Lượt đăng ký không có filter policy sẽ nhận mọi thông điệp của topic.

## Phần 3: Kinesis (Trang 469-475)

### Kinesis Data Streams (Trang 469-470)

- Thu thập và lưu trữ dữ liệu liên tục theo thời gian thực.
- Thời gian lưu dữ liệu từ **1 đến 365 ngày**; bên xử lý có thể phát lại dữ liệu còn trong thời gian lưu.
- Dữ liệu không thể xóa khỏi stream trước khi hết thời gian lưu.
- Kích thước một bản ghi tối đa **1 MB**.
- Bảo đảm thứ tự đối với dữ liệu có cùng **Partition ID**.
- Mã hóa khi lưu bằng KMS keys và mã hóa khi truyền bằng HTTPS.
- **Kinesis Producer Library (KPL)** hỗ trợ xây dựng bên gửi; **Kinesis Client Library (KCL)** hỗ trợ xây dựng bên xử lý.

### Chế độ capacity của Kinesis Data Streams (Trang 471)

| Chế độ | Đặc điểm |
|--------|----------|
| **Provisioned** | Chọn số shard; mỗi shard hỗ trợ 1 MB/giây ghi (hoặc 1.000 bản ghi/giây) và 2 MB/giây đọc; tự điều chỉnh số shard; tính phí theo shard mỗi giờ. |
| **On-demand** | Không cần chuẩn bị capacity; mức mặc định 4 MB/giây hoặc 4.000 bản ghi/giây; tự mở rộng dựa trên mức lưu lượng đỉnh đã quan sát trong 30 ngày; tính phí theo stream và lượng dữ liệu vào/ra. |

### Amazon Data Firehose (Trang 472-473)

- Tên trước đây là **Kinesis Data Firehose**.
- Dịch vụ không cần quản lý máy chủ, được AWS quản lý hoàn toàn, dùng để chuyển dữ liệu liên tục đến đích.
- Đích có thể là S3, Redshift, OpenSearch Service, dịch vụ bên thứ ba như Splunk/MongoDB/Datadog hoặc HTTP endpoint tùy chỉnh.
- Giao dữ liệu gần thời gian thực, có đệm theo kích thước hoặc thời gian.
- Hỗ trợ CSV, JSON, Parquet, Avro, văn bản thô và dữ liệu nhị phân.
- Có thể chuyển đổi sang Parquet/ORC, nén bằng gzip/snappy và biến đổi dữ liệu tùy chỉnh bằng Lambda.

### Kinesis Data Streams và Data Firehose (Trang 474)

| Đặc điểm | Kinesis Data Streams | Amazon Data Firehose |
|---------|----------------------|----------------------|
| Mục đích | Thu thập và xử lý dữ liệu liên tục | Chuyển dữ liệu đến đích |
| Quản lý | Tự xây dựng bên gửi và bên xử lý | Dịch vụ được quản lý hoàn toàn |
| Độ trễ | Thời gian thực | Gần thời gian thực |
| Capacity | Provisioned hoặc On-demand | Tự động mở rộng |
| Lưu trữ và phát lại | Lưu đến 365 ngày, có thể phát lại | Không lưu trữ stream, không hỗ trợ phát lại |

### Amazon Managed Service for Apache Flink (Trang 475)

- Trước đây có tên **Kinesis Data Analytics for Apache Flink**.
- Apache Flink là framework xử lý dữ liệu liên tục; có thể viết ứng dụng bằng Java, Scala hoặc SQL.
- Dịch vụ chạy ứng dụng Flink trên cụm được AWS quản lý, hỗ trợ tài nguyên tính toán được chuẩn bị sẵn, xử lý song song, tự động mở rộng, điểm kiểm tra và bản chụp trạng thái.
- Không đọc dữ liệu trực tiếp từ Amazon Data Firehose.

## Phần 4: So sánh SQS, SNS và Kinesis (Trang 476)

| Đặc điểm | SQS | SNS | Kinesis Data Streams |
|---------|-----|-----|----------------------|
| Mô hình | Hàng đợi | Pub/sub | Luồng dữ liệu |
| Giao nhận | Bên xử lý chủ động lấy | Topic đẩy đến bên đăng ký nhận | Bên xử lý chủ động lấy từ shard |
| Mục đích chính | Tách rời và xếp hàng công việc | Phân phối một thông điệp đến nhiều bên nhận | Thu thập và xử lý dữ liệu liên tục theo thời gian thực |
| Thứ tự | Có thể dùng FIFO | Có thể dùng SNS FIFO | Theo partition/shard |
| Phát lại dữ liệu | Không có cơ chế phát lại của stream | Không có cơ chế phát lại của stream | Có thể phát lại trong thời gian lưu |

## Bài tập trên slide (Trang 477)

- Tạo và đăng ký nhận SNS topic.
- Thêm sự kiện S3 gửi đến SNS.
