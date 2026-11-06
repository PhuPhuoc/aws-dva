# Giải thích bổ sung - Section 9: AWS Integration & Messaging

> *Phần này bổ sung kiến thức thực tế; nội dung mở rộng được tách biệt khỏi phần dịch slide.*

---

## 1. Chọn mô hình giao tiếp và tách rời ứng dụng

### Giải thích cốt lõi

Giao tiếp đồng bộ buộc bên gọi chờ phản hồi từ dịch vụ đích. Khi đích chậm hoặc tải tăng, độ trễ và lỗi có thể lan ngược về bên gọi. Tách rời bằng dịch vụ trao đổi thông điệp tạo ranh giới giữa hai bên để chúng có thể mở rộng và xử lý theo nhịp riêng.

- **SQS** phù hợp với hàng đợi công việc: nhiều bên xử lý cạnh tranh nhận việc từ hàng đợi.
- **SNS** phù hợp với mô hình xuất bản/đăng ký nhận: một lần gửi có thể thông báo cho nhiều hệ thống.
- **Kinesis Data Streams** phù hợp với luồng sự kiện cần lưu giữ và phát lại trong thời gian cấu hình.
- **Amazon Data Firehose** phù hợp khi mục tiêu chính là chuyển dữ liệu liên tục đến các đích được hỗ trợ mà không tự quản lý bên xử lý.

Các lựa chọn này khác nhau ở cách bên nhận lấy dữ liệu và khả năng phát lại; không có một dịch vụ thay thế hoàn toàn các dịch vụ còn lại.

### NÊN làm (DO)

- **Chọn dịch vụ theo cách bên nhận cần lấy dữ liệu** → Dùng SQS cho công việc cần xử lý độc lập; SNS khi nhiều hệ thống cần cùng một sự kiện; Kinesis Data Streams khi cần thứ tự theo phân vùng và phát lại. So với gọi đồng bộ trực tiếp, các mô hình này giảm phụ thuộc vào trạng thái chạy tức thời của từng bên.
- **Thiết kế bên gửi và bên xử lý có thể mở rộng riêng** → Hàng đợi hoặc luồng dữ liệu hấp thụ chênh lệch tốc độ tạm thời; bên xử lý có thể tăng số lượng theo lượng công việc tồn. So với ép tầng nhận xử lý ngay mọi đợt tăng đột biến, cách này giảm lan truyền nghẽn.

### KHÔNG NÊN làm (DON'T)

- **Dùng SNS như hàng đợi lưu công việc cho bên xử lý đang bận** → SNS phân phối thông báo nhưng không thay thế hàng đợi lưu trữ để mỗi bên xử lý tiêu thụ độc lập. NÊN: đăng ký một SQS queue cho từng bên nhận cần đệm dữ liệu và thử lại.
- **Dùng Kinesis chỉ để gửi một tác vụ đơn giản** → Kinesis đòi hỏi lựa chọn phân vùng, bên nhận và thời gian lưu; phần vận hành tăng nếu không cần ngữ nghĩa của luồng dữ liệu. NÊN: dùng SQS cho công việc độc lập hoặc SNS cho thông báo một-nhiều.

---

## 2. SQS tiêu chuẩn: giao nhận, thử lại và DLQ

### Giải thích cốt lõi

SQS tiêu chuẩn giao thông điệp theo cơ chế **at-least-once**: thông điệp có thể được giao lại. Sau khi bên xử lý nhận thông điệp, Visibility Timeout làm thông điệp tạm thời ẩn; chỉ xóa thông điệp sau khi xử lý thành công mới xác nhận hoàn tất. Nếu hết thời gian mà chưa xóa, thông điệp hiển thị lại để có thể thử lại. DLQ tách thông điệp vượt ngưỡng thử lại khỏi hàng đợi chính để điều tra, thay vì lặp vô hạn trong luồng xử lý.

Long Polling giảm phản hồi rỗng bằng cách chờ thông điệp trong một khoảng thời gian. Delay Queue trì hoãn thời điểm thông điệp xuất hiện; đây không phải cơ chế thử lại hay thay thế Visibility Timeout.

### NÊN làm (DO)

- **Đặt Visibility Timeout đủ cho thời gian xử lý dự kiến và kéo dài khi tác vụ lâu** → Thời gian quá ngắn có thể khiến bên xử lý thứ hai nhận cùng thông điệp khi bên đầu vẫn đang làm việc. So với đặt một thời gian cực dài cho mọi tác vụ, điều chỉnh theo loại công việc giúp thông điệp lỗi được xử lý lại kịp thời hơn.
- **Cấu hình `maxReceiveCount` và DLQ cho thông điệp không xử lý được** → DLQ giữ thông điệp độc hại hoặc không hợp lệ khỏi vòng thử lại liên tục, giúp điều tra rồi đưa lại sau khi sửa lỗi. So với chỉ tăng thời gian ẩn, DLQ cho biết thông điệp đã thất bại nhiều lần.
- **Làm cho thao tác của bên xử lý có tính lũy đẳng và chỉ xóa thông điệp sau khi hoàn tất** → SQS có thể giao lại thông điệp, nên tính lũy đẳng hạn chế tác động phụ bị lặp. So với giả định mỗi thông điệp chỉ đến đúng một lần, đây là cách xử lý an toàn hơn với cơ chế giao ít nhất một lần.
- **Bật Long Polling khi bên xử lý thường gặp hàng đợi rỗng** → `WaitTimeSeconds` chờ thông điệp đến thay vì liên tục gọi API và nhận phản hồi rỗng; so với thăm dò ngắn, cách này giảm số lần gọi và chi phí liên quan.

### KHÔNG NÊN làm (DON'T)

- **Xóa thông điệp trước khi ghi nhận kết quả xử lý bền vững** → Nếu bên xử lý dừng sau khi xóa, công việc mất khỏi hàng đợi dù chưa hoàn thành. NÊN: hoàn tất thao tác nghiệp vụ rồi mới gọi `DeleteMessage`; nếu có thể, lưu trạng thái theo cách lũy đẳng.
- **Đặt thời gian ẩn rất ngắn hoặc thử lại vô hạn mà không có DLQ** → Thông điệp có thể bị xử lý đồng thời hoặc chiếm bên xử lý mãi trong vòng lặp lỗi. NÊN: đặt thời gian phù hợp, giới hạn số lần nhận và chuyển thông điệp lỗi vào DLQ.
- **Dùng Delay Queue để thay thế DLQ** → Cơ chế trì hoãn chỉ hoãn khả năng nhận thông điệp; nó không phân loại thông điệp đã thất bại nhiều lần. NÊN: dùng DLQ cho thông điệp vượt ngưỡng thử lại, dùng trì hoãn cho thời điểm xử lý trong tương lai.

---

## 3. SQS FIFO: thứ tự, nhóm và khử trùng lặp

### Giải thích cốt lõi

SQS FIFO giữ thứ tự trong một `MessageGroupId` và dùng cơ chế khử trùng lặp trong cửa sổ 5 phút để loại bỏ các lần gửi lặp theo nội dung hoặc theo `MessageDeduplicationId`. Thông điệp cùng nhóm được xử lý lần lượt; các nhóm khác nhau có thể được xử lý song song. Vì thế, lựa chọn `MessageGroupId` quyết định phạm vi giữ thứ tự và mức độ song song.

So với hàng đợi tiêu chuẩn, FIFO phù hợp khi thứ tự có ý nghĩa nghiệp vụ nhưng có giới hạn thông lượng theo nội dung slide. Cơ chế khử trùng lặp của SQS không làm cho mọi thao tác bên ngoài bên xử lý trở nên tuyệt đối một lần: bên xử lý vẫn nên chịu được việc chạy lặp.

### NÊN làm (DO)

- **Chọn `MessageGroupId` theo thực thể cần giữ thứ tự, chẳng hạn mã khách hàng hoặc đơn hàng** → Các thông điệp của cùng thực thể được tuần tự hóa, còn các thực thể khác có thể chạy song song. So với một mã nhóm duy nhất cho toàn hàng đợi, cách này tận dụng xử lý song song tốt hơn.
- **Dùng FIFO khi thứ tự và khử trùng lặp quan trọng hơn thông lượng tối đa** → FIFO cung cấp thứ tự theo nhóm và khử trùng lặp trong cửa sổ quy định. So với hàng đợi tiêu chuẩn, đây là lựa chọn phù hợp hơn cho chuỗi sự kiện cần xử lý theo trình tự.
- **Vẫn làm thao tác bên xử lý có tính lũy đẳng** → Giao lại thông điệp hoặc lỗi ở hệ thống đích vẫn có thể làm thao tác nghiệp vụ lặp lại. So với chỉ dựa vào khử trùng lặp của hàng đợi, tính lũy đẳng bảo vệ cả trạng thái bên ngoài SQS.

### KHÔNG NÊN làm (DON'T)

- **Đặt mọi thông điệp vào cùng một `MessageGroupId` khi cần thông lượng cao** → Chỉ một luồng trong nhóm được xử lý tại một thời điểm, tạo nút thắt. NÊN: chia nhóm theo khóa nghiệp vụ vẫn cần giữ thứ tự.
- **Chọn FIFO khi không cần thứ tự hoặc khử trùng lặp** → Có thể chạm giới hạn thông lượng FIFO mà không nhận được lợi ích cần thiết. NÊN: dùng hàng đợi tiêu chuẩn nếu thứ tự ở mức tốt nhất có thể và giao nhận ít nhất một lần đáp ứng yêu cầu.
- **Cho rằng khử trùng lặp bảo đảm tác động nghiệp vụ chỉ xảy ra một lần tuyệt đối** → Cơ chế này có phạm vi và cửa sổ nhất định, không kiểm soát mọi tác động phụ ở bên xử lý. NÊN: lưu khóa chống lặp hoặc kiểm tra trạng thái xử lý trong hệ thống nghiệp vụ.

---

## 4. SNS phân phối một-nhiều và lọc thông điệp

### Giải thích cốt lõi

SNS topic nhận một lần gửi rồi đẩy thông điệp đến các bên đăng ký nhận. Mô hình phân phối một-nhiều phù hợp khi nhiều hệ thống cần cùng một sự kiện. Một SQS queue riêng cho mỗi luồng xử lý bổ sung khả năng đệm, tiêu thụ không đồng bộ và thử lại riêng. Chính sách lọc loại bỏ thông điệp không phù hợp trước khi gửi đến lượt đăng ký.

So với nhiều bên gửi kết nối trực tiếp đến từng bên nhận, topic giảm số kết nối cần quản lý. So với chỉ dùng một hàng đợi, mô hình xuất bản/đăng ký nhận không buộc các bên xử lý chia sẻ cùng một luồng thông điệp.

### NÊN làm (DO)

- **Dùng một SQS queue riêng cho mỗi bên nhận cần xử lý độc lập** → Một bên xử lý chậm không làm bên khác phải chờ trên cùng hàng đợi; queue giữ thông điệp cho bên đó xử lý sau. So với tất cả bên xử lý cùng thăm dò một queue, phân phối một-nhiều giữ mỗi nhánh độc lập.
- **Cấu hình quyền cho SNS gửi đến SQS và kiểm tra lượt đăng ký nhận** → Nếu queue policy không cho SNS gửi, phân phối không hoàn tất. So với chỉ cấp quyền cho bên gửi, cần kiểm tra cả chính sách của topic và queue.
- **Dùng chính sách lọc khi mỗi bên nhận chỉ cần một tập con sự kiện** → Giảm thông điệp thừa ở các hệ thống phía sau; so với để mỗi bên xử lý tự lọc sau khi nhận, lọc ở lượt đăng ký tránh xử lý không cần thiết.

### KHÔNG NÊN làm (DON'T)

- **Giả định SNS tự cung cấp hàng đợi bền vững cho bên nhận chậm** → Bên nhận không có SQS riêng phụ thuộc vào khả năng tiếp nhận của điểm cuối. NÊN: đăng ký SQS làm bên nhận nếu cần đệm, thử lại và xử lý trễ.
- **Tạo nhiều S3 notification rules trùng loại sự kiện và prefix để gửi cùng sự kiện đến nhiều SQS queues** → Slide nêu giới hạn một rule cho cùng tổ hợp loại sự kiện/prefix. NÊN: gửi sự kiện đến SNS rồi phân phối một-nhiều sang các hàng đợi SQS.
- **Bỏ qua policy và bộ lọc khi vận hành phân phối một-nhiều** → Chính sách sai làm bên nhận không nhận được thông điệp; bộ lọc sai có thể loại bỏ thông điệp cần thiết. NÊN: kiểm tra chính sách, lượt đăng ký, phạm vi lọc và thử với thông điệp đại diện.

---

## 5. Kinesis Data Streams, Data Firehose và Apache Flink

### Giải thích cốt lõi

**Kinesis Data Streams** lưu dữ liệu liên tục theo shard trong khoảng thời gian đã cấu hình, cho phép bên xử lý đọc lại dữ liệu còn lưu. Khóa phân vùng quyết định shard; dữ liệu cùng khóa giữ thứ tự. **Amazon Data Firehose** là dịch vụ chuyển dữ liệu được quản lý để đệm và gửi đến đích; dịch vụ này gần thời gian thực nhưng không phải luồng lưu trữ có khả năng phát lại. **Amazon Managed Service for Apache Flink** dùng để chạy logic xử lý dữ liệu liên tục.

So với Data Firehose, Data Streams cần lựa chọn capacity và bên xử lý nhưng cho quyền kiểm soát và phát lại. So với Data Streams, Firehose giảm công sức tự xây đường ống chuyển dữ liệu khi chỉ cần gửi đến đích được hỗ trợ.

### NÊN làm (DO)

- **Dùng Data Streams khi cần phát lại, nhiều bên xử lý hoặc độ trễ thấp** → Thời gian lưu cho phép bên xử lý đọc lại dữ liệu trong cửa sổ lưu trữ. So với Firehose, Streams phù hợp hơn khi ứng dụng cần tự quyết định cách tiêu thụ và phát lại.
- **Chọn khóa phân vùng có phân bố đủ đều** → Một khóa quá nóng có thể dồn lưu lượng vào một shard và giới hạn xử lý song song. So với dùng cùng một khóa cho mọi bản ghi, khóa phù hợp với thực thể nghiệp vụ giữ thứ tự theo thực thể mà vẫn phân tán tải.
- **Dùng Data Firehose khi yêu cầu chính là chuyển dữ liệu đến một đích được hỗ trợ** → Dịch vụ quản lý việc đệm, chuyển đổi định dạng và tích hợp Lambda. So với tự viết bên gửi, bên xử lý và đường ống chuyển dữ liệu, cách này giảm phần vận hành.
- **Dùng Managed Service for Apache Flink khi cần xử lý luồng dữ liệu liên tục** → Flink hỗ trợ xử lý song song, điểm kiểm tra và bản chụp trạng thái. So với Firehose, Flink thực thi logic xử lý chứ không chỉ chuyển dữ liệu đến đích.

### KHÔNG NÊN làm (DON'T)

- **Chọn Data Firehose khi bắt buộc phát lại dữ liệu đã gửi** → Slide mô tả Firehose không lưu luồng và không hỗ trợ phát lại. NÊN: dùng Data Streams với thời gian lưu phù hợp.
- **Chọn Data Streams mà không ước lượng thông lượng hoặc kiểm tra shard nóng** → Chế độ Provisioned cần quản lý số shard; khóa phân vùng lệch tải có thể làm giảm khả năng ghi/đọc. NÊN: ước lượng tải, chọn chế độ capacity phù hợp và theo dõi phân bố khóa.
- **Giả định Data Firehose là nguồn đọc trực tiếp của Managed Service for Apache Flink** → Slide ghi Flink không đọc từ Amazon Data Firehose. NÊN: thiết kế nguồn Flink từ dịch vụ luồng dữ liệu được hỗ trợ, chẳng hạn Kinesis Data Streams.
