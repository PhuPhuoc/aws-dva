# Section 12 - Amazon DynamoDB (Trang 611-660)

---

## Khái niệm NoSQL và DynamoDB (Trang 611-615)

### Cơ sở dữ liệu quan hệ và NoSQL (Trang 612-613)

| Đặc điểm | Cơ sở dữ liệu quan hệ (RDBMS) | NoSQL |
|---|---|---|
| Mô hình dữ liệu | Quan hệ, cần thiết kế schema chặt chẽ | Không quan hệ, phân tán |
| Truy vấn | SQL; hỗ trợ JOIN, tổng hợp và phép tính phức tạp | Ít hoặc không hỗ trợ JOIN; dữ liệu cần cho truy vấn thường được gom vào cùng item |
| Mở rộng | Có thể mở rộng dọc; mở rộng khả năng đọc ngang bằng cách thêm EC2/RDS Read Replicas | Mở rộng ngang |
| Cách thiết kế | Mô hình hóa theo quan hệ dữ liệu | Mô hình hóa theo truy vấn ứng dụng |

- Không có mô hình nào luôn đúng hơn; cần chọn cách mô hình hóa phù hợp với truy vấn và workload.
- NoSQL thường không thực hiện các phép tổng hợp như `SUM` hay `AVG` như cơ sở dữ liệu quan hệ.

### Amazon DynamoDB (Trang 614-615)

- Cơ sở dữ liệu NoSQL được quản lý hoàn toàn, có tính sẵn sàng cao và sao chép qua nhiều AZ.
- Là cơ sở dữ liệu phân tán, mở rộng cho workload lớn, độ trễ truy xuất thấp.
- Theo slide, DynamoDB có thể xử lý hàng triệu requests mỗi giây, hàng nghìn tỷ item và hàng trăm TB dữ liệu.
- Tích hợp IAM để kiểm soát bảo mật, phân quyền và quản trị.
- Hỗ trợ lập trình hướng sự kiện qua DynamoDB Streams.
- Có hai lớp bảng: Standard và Infrequent Access (IA).
- DynamoDB gồm các **table**; mỗi table có **Primary Key** phải chọn khi tạo.
- Table có thể chứa số lượng item không giới hạn; item gồm các attribute có thể thêm theo thời gian và có thể để null.
- Kích thước tối đa của một item là **400 KB**.

| Nhóm kiểu dữ liệu | Kiểu được hỗ trợ |
|---|---|
| Scalar | String, Number, Binary, Boolean, Null |
| Document | List, Map |
| Set | String Set, Number Set, Binary Set |

## Primary Key và phân vùng dữ liệu (Trang 616-618, 624)

### Các loại Primary Key (Trang 616-617)

- **Partition Key (HASH):** mỗi item phải có giá trị khóa phân vùng duy nhất. Nên chọn giá trị đa dạng để phân phối dữ liệu, ví dụ `User_ID`.
- **Partition Key + Sort Key (HASH + RANGE):** cặp giá trị phải duy nhất. Các item cùng Partition Key được nhóm lại; ví dụ bảng người dùng-trò chơi dùng `User_ID` và `Game_ID`.

### Chọn Partition Key (Trang 618)

- Với cơ sở dữ liệu phim, slide so sánh `movie_id`, `producer_name`, `leader_actor_name` và `movie_language`.
- `movie_id` có cardinality cao nhất nên phân phối dữ liệu tốt hơn.
- `movie_language` có ít giá trị và có thể lệch mạnh về một ngôn ngữ, nên dễ tạo phân phối không đều.

### Phân vùng nội bộ (Trang 624)

- DynamoDB dùng thuật toán hash trên Partition Key để xác định partition lưu item.
- Ước tính số partition theo capacity:
  - `RCUs_total / 3000 + WCUs_total / 1000`
- Ước tính theo kích thước:
  - `Total size / 10 GB`
- Tổng số partition là `ceil(max(số partition theo capacity, số partition theo kích thước))`.
- RCU và WCU được phân phối trên các partition.

## Read/Write Capacity và throttling (Trang 619-626)

### Chế độ capacity (Trang 619)

| Chế độ | Cách quản lý | Chi phí theo slide |
|---|---|---|
| **Provisioned** (mặc định) | Chỉ định read/write throughput; cần lập kế hoạch trước; có thể bật Auto Scaling | Trả cho RCU/WCU đã provision |
| **On-Demand** | Capacity tự điều chỉnh theo workload; không cần lập kế hoạch RCU/WCU | Trả theo lượt sử dụng; slide nêu chi phí cao hơn |

- Có thể đổi giữa hai chế độ, nhưng theo slide chỉ đổi được một lần mỗi 24 giờ.

### Provisioned RCU/WCU và throttling (Trang 620, 625)

- **RCU** đo throughput đọc; **WCU** đo throughput ghi.
- Có thể vượt throughput tạm thời nhờ Burst Capacity. Khi burst capacity cạn, DynamoDB có thể trả `ProvisionedThroughputExceededException`.
- Nguyên nhân throttling gồm Hot Keys, Hot Partitions và item có kích thước lớn.
- Cách xử lý trên slide: exponential backoff, phân phối Partition Key tốt hơn; nếu vấn đề là read Hot Key, cân nhắc DynamoDB Accelerator (DAX).

### Write Capacity Units (WCU) (Trang 621)

- 1 WCU đáp ứng một write mỗi giây cho item có kích thước tối đa 1 KB.
- Item lớn hơn 1 KB cần thêm WCU theo từng KB được làm tròn lên.

| Workload | Tính toán | WCU cần thiết |
|---|---|---:|
| 10 writes/giây, item 2 KB | `10 × 2 KB / 1 KB` | 20 |
| 6 writes/giây, item 4,5 KB | `6 × 5 KB / 1 KB` | 30 |
| 120 writes/phút, item 2 KB | `(120 / 60) × 2 KB / 1 KB` | 4 |

### Eventual và Strongly Consistent Read (Trang 622)

- **Eventually Consistent Read** là mặc định; đọc ngay sau khi ghi có thể nhận dữ liệu cũ do replication.
- **Strongly Consistent Read** trả về dữ liệu mới nhất theo nội dung slide; bật `ConsistentRead=true` cho `GetItem`, `BatchGetItem`, `Query` hoặc `Scan`.
- Strongly Consistent Read tiêu thụ gấp đôi RCU so với Eventually Consistent Read.

### Read Capacity Units (RCU) (Trang 623)

- 1 RCU đáp ứng một Strongly Consistent Read/giây hoặc hai Eventually Consistent Reads/giây cho item tối đa 4 KB.
- Item lớn hơn 4 KB được làm tròn lên theo bội số 4 KB để tính capacity.

| Workload | Tính toán | RCU cần thiết |
|---|---|---:|
| 10 Strongly Consistent Reads/giây, item 4 KB | `10 × 4 KB / 4 KB` | 10 |
| 16 Eventually Consistent Reads/giây, item 12 KB | `(16 / 2) × 12 KB / 4 KB` | 24 |
| 10 Strongly Consistent Reads/giây, item 6 KB | `10 × 8 KB / 4 KB` | 20 |

### On-Demand (Trang 626)

- Read/write tự điều chỉnh theo workload; không cần lập kế hoạch WCU/RCU.
- Slide ghi mức capacity là “unlimited”, không throttling và chi phí khoảng **2,5 lần** Provisioned; tính theo Read Request Units (RRU) và Write Request Units (WRU).
- Dùng cho workload chưa biết trước hoặc traffic khó dự đoán. Các con số và giới hạn này là nội dung theo slide; cần đối chiếu giới hạn AWS hiện hành khi thiết kế.

## Ghi, đọc và truy vấn dữ liệu (Trang 627-638)

### Ghi dữ liệu (Trang 627)

- `PutItem`: tạo item hoặc thay thế toàn bộ item cùng Primary Key; tiêu thụ WCU.
- `UpdateItem`: sửa attribute hoặc thêm item mới nếu chưa tồn tại; có thể dùng làm Atomic Counter.
- **Conditional Writes:** chỉ ghi/cập nhật/xóa khi điều kiện đúng; nếu không, thao tác trả lỗi. Dùng để xử lý truy cập đồng thời; slide ghi không ảnh hưởng hiệu năng.

### Đọc dữ liệu (Trang 628-630)

- `GetItem`: đọc theo Primary Key, có thể dùng Partition Key đơn hoặc cặp Partition Key + Sort Key. Mặc định eventual consistency; có thể yêu cầu strongly consistent read. `ProjectionExpression` giới hạn attribute được trả về.
- `Query` yêu cầu giá trị Partition Key bằng toán tử `=`; có thể thêm điều kiện Sort Key như `=`, `<=`, `>`, `>=`, `Between`, `Begins with`.
  - `FilterExpression` lọc thêm theo non-key attributes; không dùng để lọc Partition Key/Sort Key.
  - Kết quả bị giới hạn bởi `Limit` hoặc tối đa 1 MB mỗi lần trả về; hỗ trợ pagination.
  - Có thể query table, LSI hoặc GSI.
- `Scan` đọc toàn bộ table rồi mới lọc; tốn nhiều RCU và kém hiệu quả hơn Query.
  - Mỗi lần trả về tối đa 1 MB; dùng pagination để đọc tiếp.
  - Có thể giới hạn tác động bằng `Limit`, giảm kích thước kết quả hoặc tạm dừng.
  - Parallel Scan chia các segment cho nhiều worker, tăng throughput nhưng cũng tăng RCU tiêu thụ.
  - `ProjectionExpression` và `FilterExpression` không làm giảm RCU theo slide.

### Xóa dữ liệu (Trang 631)

- `DeleteItem` xóa một item và có thể kèm điều kiện.
- `DeleteTable` xóa toàn bộ table và các item; nhanh hơn gọi `DeleteItem` cho từng item.

### Batch Operations (Trang 632)

- Gộp nhiều thao tác để giảm số API calls; các thao tác trong batch được thực hiện song song.
- Một phần batch có thể thất bại; cần gửi lại các item/key chưa xử lý.
- `BatchWriteItem`: tối đa 25 `PutItem`/`DeleteItem`, tối đa 16 MB mỗi request và 400 KB mỗi item; không hỗ trợ update. Các thao tác chưa hoàn tất nằm trong `UnprocessedItems`.
- `BatchGetItem`: tối đa 100 items và 16 MB; các item được đọc song song. Key chưa hoàn tất nằm trong `UnprocessedKeys`.
- Với unprocessed items/keys, dùng exponential backoff hoặc điều chỉnh RCU/WCU phù hợp.

### PartiQL và Conditional Writes (Trang 633-638, 643-644)

- PartiQL cung cấp cú pháp kiểu SQL để `SELECT`, `INSERT`, `UPDATE`, `DELETE` dữ liệu DynamoDB; có thể chạy qua Console, NoSQL Workbench, API, CLI hoặc SDK, hỗ trợ batch operations.
- PartiQL có thể chạy truy vấn trên nhiều DynamoDB tables.
- Slide Trang 634 liệt kê `BatchWriteItem` trong nhóm thao tác có thể dùng điều kiện ghi.
- **Đính chính kỹ thuật:** API `BatchWriteItem` hiện hành không hỗ trợ `ConditionExpression`; dùng các thao tác `PutItem`/`UpdateItem`/`DeleteItem` riêng lẻ hoặc `TransactWriteItems` khi cần điều kiện.
- `FilterExpression` lọc kết quả đọc; **Condition Expression** quyết định một thao tác ghi có được phép thực hiện hay không.
- Có thể dùng `attribute_not_exists` để tránh ghi đè item/khóa đã tồn tại.
- **Optimistic Locking:** dùng một attribute làm version number; kiểm tra version hiện tại bằng conditional write trước khi cập nhật hoặc xóa.

## Indexes và tăng tốc đọc (Trang 639-646)

### LSI và GSI (Trang 640-642)

| Đặc điểm | LSI | GSI |
|---|---|---|
| Khóa | Giữ Partition Key của table, thay Sort Key | Primary Key khác, có thể là Partition Key hoặc Partition Key + Sort Key |
| Số lượng/cấu hình | Tối đa 5; phải khai báo khi tạo table | Có thể thêm hoặc chỉnh sau khi tạo table |
| Capacity | Dùng RCU/WCU của table chính | Phải provision RCU/WCU riêng cho index |
| Lưu ý | Có projections `KEYS_ONLY`, `INCLUDE`, `ALL` | Có projections `KEYS_ONLY`, `INCLUDE`, `ALL`; write throttling ở GSI có thể làm table chính bị throttling |

- Index key dùng kiểu scalar String, Number hoặc Binary.
- Dùng index để query theo khóa thay thế hoặc non-key attributes.

### PartiQL và Optimistic Locking (Trang 643-644)

- PartiQL hỗ trợ một phần các lệnh SQL như `INSERT`, `UPDATE`, `SELECT`, `DELETE` và batch operations.
- Optimistic Locking dùng Conditional Writes và version attribute để phát hiện item đã bị thay đổi trước khi ghi/xóa.

### DynamoDB Accelerator (DAX) (Trang 645-646)

- DAX là cache in-memory được quản lý hoàn toàn, có tính sẵn sàng cao và tích hợp với API DynamoDB.
- Cache reads/queries có độ trễ microsecond; giúp giảm tải đọc cho Hot Key mà không cần viết logic cache riêng trong ứng dụng.
- TTL mặc định theo slide là 5 phút; cluster tối đa 10 nodes; slide khuyến nghị ít nhất 3 nodes cho production multi-AZ.
- Hỗ trợ bảo mật như mã hóa at rest bằng KMS, VPC, IAM và CloudTrail.

> *Sơ đồ (Trang 646): So sánh DynamoDB Accelerator (DAX) và ElastiCache.*

## Streams, TTL và giao dịch (Trang 647-655)

### DynamoDB Streams (Trang 647-650)

- Stream lưu chuỗi có thứ tự các thay đổi item-level: create, update, delete; dữ liệu được giữ tối đa 24 giờ.
- Có thể chuyển record đến Kinesis Data Streams, đọc bằng Lambda hoặc Kinesis Client Library.
- Use cases: phản ứng theo thời gian thực, analytics, cập nhật bảng dẫn xuất, ghi OpenSearch Service hoặc replication giữa Region.
- Chọn loại dữ liệu ghi vào stream:
  - `KEYS_ONLY`: chỉ các khóa của item đã thay đổi.
  - `NEW_IMAGE`: toàn bộ item sau thay đổi.
  - `OLD_IMAGE`: toàn bộ item trước thay đổi.
  - `NEW_AND_OLD_IMAGES`: cả trạng thái trước và sau.
- Streams gồm shards do AWS tự quản lý; record không được tạo hồi tố khi bật stream.
- Để đọc stream bằng Lambda, cấu hình Event Source Mapping và cấp quyền phù hợp. Lambda được gọi đồng bộ.

### Time To Live (TTL) (Trang 651)

- TTL tự động xóa item sau thời điểm hết hạn; attribute TTL phải là Number chứa Unix Epoch timestamp.
- Theo slide, xóa không tiêu thụ WCU và item hết hạn được xóa trong vòng 48 giờ.
- Item đã hết hạn nhưng chưa bị xóa vẫn xuất hiện trong reads/queries/scans; lọc bỏ nếu không muốn trả về.
- TTL xóa item khỏi LSI/GSI; thao tác xóa được ghi vào DynamoDB Streams.
- Dùng để giảm dữ liệu cũ hoặc đáp ứng yêu cầu lưu trữ dữ liệu.

### AWS CLI pagination (Trang 652)

- `--projection-expression`: chọn attribute cần trả về.
- `--filter-expression`: lọc dữ liệu trả về.
- `--page-size`: điều chỉnh số item AWS CLI lấy mỗi API call; slide nêu mặc định là 1.000 items, và có thể cần nhiều API calls hơn.
- `--max-items`: giới hạn số item hiển thị và trả về `NextToken`.
- `--starting-token`: dùng `NextToken` trước đó để lấy trang tiếp theo.

### Transactions (Trang 653-655)

- Transaction phối hợp nhiều thao tác thêm/cập nhật/xóa trên một hoặc nhiều table theo kiểu **all-or-nothing**, với thuộc tính ACID.
- Read modes: eventual consistency, strong consistency, transactional. Write modes: standard hoặc transactional.
- Transactional read/write tiêu thụ gấp đôi RCU/WCU chuẩn do mỗi item có bước prepare và commit.
- `TransactGetItems` nhóm một hoặc nhiều `GetItem`; `TransactWriteItems` nhóm `PutItem`, `UpdateItem`, `DeleteItem`.
- Ví dụ theo slide: 3 transactional writes/giây cho item 5 KB cần 30 WCU; 5 transactional reads/giây cho item 5 KB làm tròn lên 8 KB cần 20 RCU.

> *Sơ đồ (Trang 654): Giao dịch được ghi lên cả hai table hoặc không ghi lên table nào.*

## Mẫu sử dụng, vận hành và bảo mật (Trang 656-660)

### Session state và loại dữ liệu (Trang 656-658)

- DynamoDB có thể lưu session state; cả DynamoDB và ElastiCache đều là key/value store, nhưng ElastiCache ở bộ nhớ còn DynamoDB là dịch vụ serverless.
- EFS cần được mount lên EC2; EBS và Instance Store phục vụ cache cục bộ, không phải cache chia sẻ; S3 có độ trễ cao hơn và không tối ưu cho object nhỏ.

> *Sơ đồ (Trang 657): Các kiểu ghi dữ liệu trong DynamoDB.*
>
> *Sơ đồ (Trang 658): Mẫu lưu trữ object kích thước lớn cùng DynamoDB.*

### Dọn và sao chép table (Trang 659)

- Dọn table:
  - Scan rồi gọi `DeleteItem` từng item: chậm, tốn RCU/WCU và chi phí.
  - Xóa table rồi tạo lại: nhanh và hiệu quả hơn khi có thể bỏ toàn bộ dữ liệu.
- Sao chép table:
  - AWS Data Pipeline.
  - Backup rồi restore sang table mới; cần thời gian.
  - Tự viết code dùng Scan + `PutItem` hoặc `BatchWriteItem`.

### Bảo mật và tính năng khác (Trang 660)

- VPC Endpoint cho phép truy cập DynamoDB mà không đi qua Internet.
- IAM kiểm soát quyền truy cập; mã hóa at rest bằng AWS KMS và in transit bằng SSL/TLS.
- Backup/Restore hỗ trợ Point-in-time Recovery (PITR) mà theo slide không ảnh hưởng hiệu năng.
- Global Tables là hệ thống multi-Region, multi-active và tự replication.
- DynamoDB Local hỗ trợ phát triển/kiểm thử ứng dụng cục bộ, không cần gọi dịch vụ DynamoDB trên Internet.
- AWS Database Migration Service (AWS DMS) có thể migration dữ liệu sang DynamoDB từ MongoDB, Oracle, MySQL, S3 và các nguồn khác.

## Bài tập trên slide (Trang 661)

- Dùng sự kiện từ S3 để kích hoạt SQS, sau đó thêm thông tin ảnh vào DynamoDB table.
