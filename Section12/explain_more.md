# Giải thích bổ sung - Section 12: Amazon DynamoDB

> *Phần này bổ sung hướng dẫn thiết kế và vận hành; nội dung mở rộng được tách khỏi phần dịch slide.*

---

### 1. Thiết kế NoSQL và khóa phân vùng

#### Giải thích cốt lõi

DynamoDB phân phối item qua các partition dựa trên hash của Partition Key. Vì vậy key quyết định cả nơi lưu dữ liệu lẫn khả năng phân phối tải. Giá trị có cardinality cao và phân bố đều giúp tránh hot partition; key ít giá trị hoặc lệch phân bố có thể dồn phần lớn request vào một partition. Composite key giữ các item liên quan chung Partition Key và dùng Sort Key để truy vấn một nhóm con.

Khác với RDBMS, DynamoDB không tối ưu cho JOIN/aggregation tùy ý. Ứng dụng cần xác định access patterns trước rồi chọn Partition Key, Sort Key và index để truy vấn trực tiếp. Mỗi item tối đa 400 KB, nên dữ liệu lớn hoặc quan hệ cần truy vấn linh hoạt có thể phù hợp dịch vụ khác.

#### NÊN làm (DO)

- **Liệt kê các truy vấn cần hỗ trợ trước khi chọn Primary Key** → Khóa quyết định cách đọc hiệu quả và phân phối request; so với thiết kế schema kiểu quan hệ rồi thêm truy vấn sau, access-pattern-first giảm nhu cầu Scan.
- **Chọn Partition Key có cardinality cao và phân bố đều** → Lưu lượng được trải trên nhiều partition; so với key như trạng thái/loại có ít giá trị, giảm nguy cơ hot key và throttling.
- **Dùng Sort Key để nhóm dữ liệu cần truy vấn cùng Partition Key** → Query có thể lấy đúng tập item theo khoảng hoặc tiền tố; so với Scan toàn table, giảm lượng dữ liệu phải đọc.
- **Để object lớn ở S3 và giữ key/metadata trong DynamoDB khi phù hợp** → Tránh vượt giới hạn item 400 KB; so với nhét blob vào item, object storage phù hợp hơn cho tệp lớn.

#### KHÔNG NÊN làm (DON'T)

- **Chọn Partition Key có ít giá trị hoặc phân bố lệch** → Một vài partition có thể thành hot partition và bị throttling dù capacity tổng còn dư. NÊN: dùng khóa có cardinality/phân bố tốt hoặc thiết kế composite key phù hợp.
- **Dùng DynamoDB như RDBMS để chạy JOIN/aggregation tùy ý** → DynamoDB không hỗ trợ mô hình truy vấn quan hệ như vậy; NÊN: thiết kế item theo access pattern, hoặc chọn RDBMS cho truy vấn quan hệ linh hoạt.
- **Lưu file lớn nguyên khối trong item** → Item vượt 400 KB không được chấp nhận và việc đọc/ghi object lớn tốn capacity. NÊN: lưu object ở S3, chỉ lưu metadata và object key trong DynamoDB.

---

### 2. Chọn capacity và tính RCU/WCU

#### Giải thích cốt lõi

Provisioned mode đặt trước read/write capacity; phù hợp workload tương đối dự đoán được và có thể kết hợp Auto Scaling. On-Demand tính theo request, tự điều chỉnh capacity và giảm công việc dự báo, nhưng thường tốn hơn khi traffic ổn định cao. Cả hai mode vẫn chịu service quotas và giới hạn phân phối; On-Demand không phải cam kết vô hạn hay miễn nhiễm throttling.

RCU/WCU được tính theo kích thước item làm tròn lên từng 4 KB cho read và 1 KB cho write, cùng consistency mode và tần suất request. Transactional operations tiêu thụ gấp đôi capacity tiêu chuẩn theo slide. Công thức ước lượng hữu ích khi lập kế hoạch nhưng không thay thế việc theo dõi partition, index và request thực tế.

#### NÊN làm (DO)

- **Dùng Provisioned kèm Auto Scaling cho workload có mức sử dụng nền tương đối ổn định** → Capacity được điều chỉnh quanh nhu cầu và thường tiết kiệm hơn so với trả request rate On-Demand liên tục.
- **Dùng On-Demand khi traffic chưa biết hoặc biến động khó dự báo** → Không phải dự báo RCU/WCU trước; so với provision cố định, giảm rủi ro thiếu capacity do sai dự báo, dù cần theo dõi chi phí và quota.
- **Tính item size đã làm tròn và consistency trước khi provision** → Item 4,5 KB cần tính như 5 KB khi ghi; strongly consistent read cần capacity gấp đôi eventual read. So với lấy kích thước trung bình chưa làm tròn, ước lượng này tránh thiếu capacity.
- **Theo dõi consumed capacity, throttles và phân bố tải trên table/index** → Capacity tổng đủ không loại trừ hot partition hay GSI throttling; so với chỉ nhìn RCU/WCU tổng, metrics giúp xác định đúng nút nghẽn.

#### KHÔNG NÊN làm (DON'T)

- **Hiểu On-Demand là không thể throttling hoặc không có giới hạn** → Quota, giới hạn tăng tải nhanh và phân bố hot key vẫn có thể gây throttling. NÊN: kiểm tra service quotas, ramp traffic có kiểm soát và xử lý lỗi throttling.
- **Provision theo average rồi bỏ qua peak/burst** → Peak có thể vượt capacity hoặc burst allowance và gây `ProvisionedThroughputExceededException`. NÊN: tính peak, bật Auto Scaling phù hợp và dùng backoff.
- **Retry nóng liên tục không có backoff** → Tăng áp lực lên partition đang nghẽn và kéo dài sự cố. NÊN: exponential backoff có jitter và chỉ retry các request chưa thành công.

---

### 3. Query, Scan và pagination

#### Giải thích cốt lõi

`Query` yêu cầu giá trị Partition Key và có thể giới hạn Sort Key; đây là lựa chọn truy vấn theo khóa. `Scan` đọc qua các item của table/index rồi mới áp dụng filter, nên thường tiêu tốn capacity lớn hơn. `FilterExpression` chỉ quyết định item nào được trả về sau khi đọc; nó không thay thế điều kiện khóa và không làm giảm capacity đã dùng để đọc dữ liệu. Query/Scan có thể bị giới hạn 1 MB mỗi response, nên ứng dụng phải tiếp tục pagination.

`GetItem` phù hợp khi đã biết Primary Key. `ProjectionExpression` giảm dữ liệu trả về qua mạng nhưng không đồng nghĩa với giảm RCU theo slide. Dùng index khi access pattern cần query theo khóa khác, thay vì Scan lặp lại.

#### NÊN làm (DO)

- **Dùng `GetItem` khi đã biết Primary Key và `Query` khi cần lấy tập item theo Partition Key** → DynamoDB đọc theo key; so với Scan, giảm công việc không cần thiết.
- **Dùng FilterExpression cho điều kiện phụ sau khi đã chọn tập khóa phù hợp** → Filter không giảm capacity đọc; so với lọc toàn bảng, chọn key trước giúp tránh đọc quá nhiều item.
- **Xử lý pagination bằng `LastEvaluatedKey`/token cho đến khi hết trang** → Một response không nhất thiết chứa mọi kết quả; so với giả định một request trả đủ dữ liệu, pagination tránh mất item ở tập kết quả lớn.
- **Dùng Parallel Scan chỉ cho tác vụ phù hợp và giới hạn tốc độ** → Nhiều segment có thể hoàn tất nhanh hơn nhưng tăng RCU; so với Scan tuần tự, có lợi về thời gian đổi lấy tải/chi phí cao hơn.

#### KHÔNG NÊN làm (DON'T)

- **Dùng Scan cho truy vấn online thường xuyên khi có thể Query theo khóa** → Scan đọc rộng và tiêu tốn RCU, có thể ảnh hưởng workload khác. NÊN: thiết kế key hoặc index theo access pattern.
- **Cho rằng FilterExpression làm giảm RCU đã tiêu thụ** → Filter chỉ bỏ item khỏi kết quả sau khi đọc; NÊN: thu hẹp bằng Partition Key/Sort Key hoặc index.
- **Bỏ qua pagination với kết quả lớn hơn giới hạn response** → Chỉ xử lý trang đầu sẽ thiếu dữ liệu. NÊN: lặp đến khi không còn continuation key/token.

---

### 4. Indexes, Conditional Writes và batch

#### Giải thích cốt lõi

LSI giữ Partition Key của table và cung cấp Sort Key khác; phải khai báo lúc tạo table, tối đa 5 và dùng capacity của table. GSI có thể dùng khóa khác, thêm sau khi tạo table nhưng cần provision capacity riêng; ghi vào GSI bị throttling có thể làm ghi table chính bị ảnh hưởng.

Conditional Writes kiểm tra điều kiện ngay trong thao tác ghi, giúp ngăn ghi đè hoặc cập nhật dựa trên trạng thái cũ. Optimistic locking thường lưu version attribute và chỉ ghi nếu version vẫn khớp. Batch operations giảm số API calls, nhưng không phải một transaction nguyên tử: một phần có thể thành công, phần còn lại được trả về dưới dạng unprocessed keys/items.

#### NÊN làm (DO)

- **Chọn LSI khi cần Sort Key thay thế nhưng vẫn truy vấn theo cùng Partition Key và biết trước lúc tạo table** → Dùng chung capacity với table; so với GSI, không cần capacity riêng nhưng kém linh hoạt sau khi tạo.
- **Chọn GSI cho access pattern cần Partition Key khác hoặc cần thêm index sau này** → Có thể thêm sau khi tạo table; so với LSI linh hoạt hơn nhưng phải cấp capacity và theo dõi throttling riêng.
- **Dùng Condition Expression/version check cho cập nhật đồng thời hoặc thao tác create-only** → Chỉ một điều kiện hợp lệ được ghi; so với read rồi write không điều kiện, giảm nguy cơ lost update.
- **Retry riêng `UnprocessedItems`/`UnprocessedKeys` bằng exponential backoff** → Batch có thể thành công một phần; so với gửi lại toàn bộ batch, retry phần chưa xử lý tránh lặp thao tác đã thành công.

#### KHÔNG NÊN làm (DON'T)

- **Tạo GSI mà không tính write capacity và phân bố Partition Key của index** → GSI có thể bị throttling và làm nghẽn ghi trên table; NÊN: lập capacity cho index, chọn key phân bố đều và giám sát riêng.
- **Coi BatchWriteItem là transaction, hoặc cho rằng nó hỗ trợ UpdateItem/ConditionExpression** → BatchWriteItem không nguyên tử, không hỗ trợ update hay conditional writes. NÊN: dùng `TransactWriteItems` cho all-or-nothing, hoặc thao tác ghi riêng lẻ có điều kiện khi phù hợp.
- **Ghi item đồng thời mà không có condition khi dữ liệu cần kiểm soát phiên bản** → Request sau có thể ghi đè thay đổi trước; NÊN: dùng optimistic locking/conditional write.

---

### 5. Transactions và tính nhất quán

#### Giải thích cốt lõi

DynamoDB Transactions nhóm thao tác đọc/ghi trên nhiều item hoặc table theo kiểu all-or-nothing, với thuộc tính ACID. `TransactGetItems` đọc và `TransactWriteItems` ghi nhiều item trong cùng một transaction. Đổi lại, transactional reads/writes tiêu thụ khoảng gấp đôi capacity tiêu chuẩn theo slide vì có bước prepare/commit. Strongly consistent read cũng tốn capacity hơn eventual read, nên mức nhất quán cần theo đúng yêu cầu nghiệp vụ.

#### NÊN làm (DO)

- **Dùng transaction khi nhiều item phải thay đổi cùng nhau để giữ invariant nghiệp vụ** → Tất cả cùng commit hoặc không item nào được cập nhật; so với các request độc lập, tránh trạng thái nửa hoàn tất.
- **Dùng Conditional Write nếu chỉ cần bảo vệ một item hoặc một cập nhật có điều kiện** → Ít thao tác hơn transaction đa item; so với transaction, tránh chi phí và phạm vi không cần thiết.
- **Tính capacity/chi phí gấp đôi cho transactional operations** → Transaction có overhead prepare/commit; so với tính như read/write thường, dự toán không bị thiếu.

#### KHÔNG NÊN làm (DON'T)

- **Dùng transaction cho mọi thao tác đơn lẻ** → Transactional capacity đắt hơn và tăng độ phức tạp; NÊN: dùng thao tác đơn hoặc conditional write nếu chỉ một item cần cập nhật.
- **Dùng nhiều thao tác riêng khi cần all-or-nothing giữa các item** → Lỗi giữa chừng để lại trạng thái một phần; NÊN: gom vào `TransactWriteItems`.

---

### 6. DAX, Streams và TTL

#### Giải thích cốt lõi

DAX là cache in-memory tương thích API DynamoDB, hữu ích cho read-heavy workload và hot keys; dữ liệu trong cache có TTL nên có thể cũ trong khoảng thời gian đó. ElastiCache là cache tổng quát hơn, thường đòi hỏi ứng dụng quản lý key, cache invalidation và luồng đọc/ghi. Chọn DAX khi workload tập trung vào đọc DynamoDB và muốn tích hợp gần với API hiện có; chọn ElastiCache khi cần cache cấu trúc/logic khác.

DynamoDB Streams ghi lại thay đổi item-level theo thứ tự trong thời gian lưu giữ 24 giờ. Lambda đọc stream qua Event Source Mapping; batch lỗi và retry cần được thiết kế theo semantics của consumer. TTL chỉ đánh dấu thời điểm hết hạn để dịch vụ dọn sau; việc xóa không đồng bộ với đúng thời điểm timestamp, nên item hết hạn có thể vẫn đọc được trước khi bị xóa.

#### NÊN làm (DO)

- **Dùng DAX cho read-heavy DynamoDB workload có hot keys và chấp nhận TTL cache** → Giảm latency/tải đọc mà ít thay đổi logic ứng dụng; so với ElastiCache, không cần tự xây lớp cache theo API DynamoDB.
- **Dùng ElastiCache khi cần cache ngoài phạm vi API DynamoDB hoặc điều khiển cache riêng** → Hỗ trợ mẫu cache tổng quát; so với DAX, linh hoạt hơn nhưng ứng dụng phải quản lý cache semantics.
- **Dùng Streams cho xử lý thay đổi theo sự kiện, và làm handler lũy đẳng** → Record có retention hữu hạn và có thể được xử lý lại; so với polling table liên tục, Streams cung cấp luồng thay đổi chuyên dụng.
- **Dùng TTL cho dữ liệu có thể hết hạn trễ và kiểm tra timestamp trong truy vấn nếu cần** → Cleanup bất đồng bộ; so với tự quét/xóa thường xuyên, giảm công việc dọn nhưng không đảm bảo item biến mất đúng thời điểm.

#### KHÔNG NÊN làm (DON'T)

- **Dùng DAX nếu mọi read phải thấy dữ liệu mới nhất ngay lập tức** → Cache có thể trả dữ liệu cũ trong TTL; NÊN: dùng đường đọc phù hợp với yêu cầu consistency hoặc giảm/điều chỉnh caching.
- **Coi TTL là cơ chế xóa chính xác tại thời điểm expiry** → Item hết hạn có thể còn xuất hiện trong read/query/scan; NÊN: kiểm tra expiry trong logic trả kết quả nếu cần loại bỏ ngay.
- **Bỏ qua retry/idempotency khi xử lý stream** → Consumer có thể nhận lại record/batch khi lỗi; NÊN: thiết kế xử lý duplicate an toàn và theo dõi iterator age/failure.

---

### 7. Bảo mật, backup và vận hành table

#### Giải thích cốt lõi

IAM kiểm soát quyền gọi API DynamoDB; VPC Endpoint giữ truy cập từ VPC trên mạng AWS. Encryption at rest dùng KMS, còn TLS bảo vệ dữ liệu trên đường truyền. Point-in-time Recovery hỗ trợ khôi phục table về thời điểm trước đó; Global Tables cung cấp replication đa Region cho workload cần đọc/ghi gần nhiều khu vực.

Xóa table rồi tạo lại nhanh hơn Scan + DeleteItem nhưng xóa toàn bộ dữ liệu và có thể phá cấu hình/consumer phụ thuộc. Sao chép table có thể qua backup/restore hoặc quy trình migration; cách phù hợp tùy yêu cầu downtime, consistency và kích thước dữ liệu.

#### NÊN làm (DO)

- **Cấp IAM quyền tối thiểu và bật mã hóa/backup theo yêu cầu dữ liệu** → Giảm phạm vi truy cập và hỗ trợ khôi phục; so với chỉ dựa vào mã hóa mặc định, policy/backup có chủ đích đáp ứng rõ yêu cầu vận hành.
- **Dùng VPC Endpoint khi workload trong VPC cần truy cập riêng DynamoDB** → Không cần định tuyến qua Internet; so với NAT/Internet egress, giữ đường truy cập dịch vụ riêng tư hơn.
- **Dùng Global Tables khi ứng dụng thật sự cần active-active đa Region** → Replication cho phép phục vụ gần người dùng nhiều Region; so với một table trong một Region, đổi lại phải xử lý thiết kế nhất quán/ghi đồng thời giữa Region.
- **Chỉ xóa và tạo lại table khi mục tiêu là reset toàn bộ dữ liệu và đã kiểm tra phụ thuộc** → Nhanh hơn xóa từng item; so với Scan + DeleteItem, giảm thời gian nhưng cần bảo vệ khỏi mất dữ liệu ngoài ý muốn.

#### KHÔNG NÊN làm (DON'T)

- **Xóa table để dọn một phần dữ liệu** → Mọi item và cấu hình table đều mất; NÊN: dùng điều kiện/xóa theo partition hoặc quy trình cleanup phù hợp.
- **Dùng Scan + DeleteItem hàng loạt mà không giới hạn throughput** → Tốn RCU/WCU, lâu và có thể ảnh hưởng traffic ứng dụng; NÊN: cân nhắc TTL, backup/restore hoặc quy trình batch có kiểm soát.
- **Bật Global Tables chỉ để thay backup/restore** → Replication đa Region không thay thế bản sao lưu khôi phục lịch sử; NÊN: duy trì PITR/backup phù hợp với mục tiêu phục hồi.
