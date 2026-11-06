# Câu hỏi ôn tập DVA-C02 - Section 12: Amazon DynamoDB

> *Câu hỏi và lựa chọn bằng tiếng Anh; phần giải thích bằng tiếng Việt.*

---

## 1. Chọn Partition Key phân phối dữ liệu

### Question 1:
A team is designing a DynamoDB table for movies. The table will receive high write traffic, and the team wants to **distribute items evenly** across partitions. Which attribute is the best partition key?

```
A. movie_language
B. producer_name
C. movie_id
D. release_month
```

### ✅ Answer: **C. movie_id**

### 📖 Explanation (Vietnamese):

**Tại sao chọn C:**
- `movie_id` thường có cardinality cao, giúp hash phân phối item trên nhiều partition.
- Key có nhiều giá trị và phân bố đều giảm nguy cơ một partition nhận phần lớn traffic.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| A | Ngôn ngữ phim thường có ít giá trị và có thể lệch mạnh về một ngôn ngữ. |
| B | Nhà sản xuất có thể có số lượng giá trị thấp hơn và workload có thể tập trung ở một số producer. |
| D | Tháng phát hành chỉ có ít giá trị lặp lại, dễ phân bố không đều. |

### 📝 Vocabulary Analysis:
- **partition key**: khóa DynamoDB dùng để hash và phân phối item.
- **distribute evenly**: phân phối đồng đều.
- **cardinality**: số lượng giá trị khác nhau của một thuộc tính.

### 🔄 Context Variations:
- **If items must be grouped by user and ordered by time** → Use `user_id` as the partition key and a timestamp as the sort key.
- **If a key has very high cardinality but traffic is still skewed** → Review actual request distribution; cardinality alone does not guarantee even traffic.

---

## 2. Chọn Query thay vì Scan

### Question 2:
An application must retrieve all orders for a specific customer, ordered by order date. The table has a partition key `customer_id` and a sort key `order_date`. Which operation should the application use?

```
A. Scan the entire table and filter by customer_id
B. Query with customer_id and a sort-key condition
C. GetItem without the full primary key
D. DeleteItem and recreate the matching orders
```

### ✅ Answer: **B. Query with customer_id and a sort-key condition**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- `Query` yêu cầu giá trị Partition Key; có thể thêm điều kiện trên Sort Key để lấy orders của customer theo khoảng thời gian.
- `Scan` đọc toàn table rồi mới lọc, tốn RCU hơn khi chỉ cần một nhóm item.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| A | Scan không tận dụng access pattern theo Primary Key và đọc dữ liệu không cần thiết. |
| C | `GetItem` cần đầy đủ Primary Key để lấy một item, không lấy danh sách orders. |
| D | Thao tác xóa/tạo lại không phục vụ truy vấn và có nguy cơ mất dữ liệu. |

### 📝 Vocabulary Analysis:
- **retrieve**: truy xuất dữ liệu.
- **sort-key condition**: điều kiện lọc theo Sort Key, như khoảng ngày.
- **entire table**: toàn bộ table.

### 🔄 Context Variations:
- **If the application knows the complete primary key for one order** → Use `GetItem`.
- **If the query cannot be modeled with an existing key** → Add an appropriate index rather than relying on frequent full-table scans.

---

## 3. Tính Write Capacity Units

### Question 3:
A DynamoDB workload writes **6 items per second**, and each item is **4.5 KB**. How many provisioned WCUs are required for the writes, before any additional safety margin?

```
A. 6 WCUs
B. 24 WCUs
C. 27 WCUs
D. 30 WCUs
```

### ✅ Answer: **D. 30 WCUs**

### 📖 Explanation (Vietnamese):

**Tại sao chọn D:**
- Một WCU hỗ trợ một write/giây cho item tối đa 1 KB.
- 4,5 KB được làm tròn lên thành 5 KB; cần `6 × 5 = 30` WCUs.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| A | Bỏ qua kích thước item, chỉ tính số writes. |
| B | Dùng 4 KB thay vì làm tròn 4,5 KB lên 5 KB. |
| C | Nhân với 4,5 KB chưa làm tròn, không đúng cách tính capacity theo slide. |

### 📝 Vocabulary Analysis:
- **provisioned WCUs**: Write Capacity Units được cấp trước.
- **safety margin**: phần capacity dự phòng cho biến động tải.
- **round up**: làm tròn lên.

### 🔄 Context Variations:
- **If the same item size is written 10 times per second** → `10 × 5 = 50` WCUs.
- **If each item is 1 KB or smaller** → One WCU per write per second is sufficient under the slide's calculation.

---

## 4. Tính Read Capacity Units

### Question 4:
A table must serve **16 eventually consistent reads per second**. Each item is **12 KB**. How many RCUs are required, before any additional safety margin?

```
A. 12 RCUs
B. 16 RCUs
C. 24 RCUs
D. 48 RCUs
```

### ✅ Answer: **C. 24 RCUs**

### 📖 Explanation (Vietnamese):

**Tại sao chọn C:**
- 12 KB cần `12 / 4 = 3` read units cho mỗi strongly consistent read.
- Eventual consistency cho phép hai reads/giây trên mỗi RCU, nên cần `(16 / 2) × 3 = 24` RCUs.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| A | Chưa tính đủ 3 khối 4 KB cho item 12 KB và tỉ lệ read. |
| B | Bỏ qua kích thước item và eventual-read factor. |
| D | Tính như strongly consistent reads, không chia đôi theo eventual consistency. |

### 📝 Vocabulary Analysis:
- **eventually consistent read**: read có thể thấy dữ liệu trễ do replication.
- **read capacity**: capacity được tính theo kích thước item và mức consistency.
- **safety margin**: capacity dự phòng.

### 🔄 Context Variations:
- **If reads must be strongly consistent** → The same request needs `16 × 3 = 48` RCUs.
- **If item size is 4 KB and reads are eventual** → One RCU supports two reads per second.

---

## 5. Chọn capacity mode

### Question 5:
A new application has **unpredictable traffic**, and the team cannot estimate its read and write capacity in advance. Which DynamoDB capacity mode is the most appropriate starting point?

```
A. Provisioned capacity with a fixed low limit
B. On-Demand capacity
C. Disable all read and write capacity
D. Use a table Scan for every request
```

### ✅ Answer: **B. On-Demand capacity**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- On-Demand tự điều chỉnh read/write capacity theo workload và không yêu cầu dự báo RCU/WCU trước.
- Cần theo dõi chi phí, quota và throttling; On-Demand không có nghĩa là không giới hạn tuyệt đối.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| A | Capacity cố định thấp có thể throttling khi traffic biến động; đây không phải lựa chọn phù hợp khi chưa biết tải. |
| C | Table vẫn cần capacity để xử lý request. |
| D | Scan là kiểu truy vấn, không phải capacity mode, và thường đọc dữ liệu không cần thiết. |

### 📝 Vocabulary Analysis:
- **unpredictable traffic**: lưu lượng khó dự đoán.
- **capacity mode**: cách quản lý throughput đọc/ghi.
- **starting point**: lựa chọn ban đầu.

### 🔄 Context Variations:
- **If traffic becomes stable and predictable** → Compare provisioned capacity with Auto Scaling against On-Demand cost.
- **If requests are throttled despite On-Demand** → Check quotas, hot keys, sudden traffic growth and index capacity.

---

## 6. Xử lý throttling do Hot Key

### Question 6:
A DynamoDB table has sufficient total provisioned capacity, but requests for one popular product are throttled. The product ID is the partition key. What is the best first response?

```
A. Retry continuously without a delay
B. Use exponential backoff and review the hot-key access pattern
C. Increase the table's item size
D. Replace all Query operations with Scan
```

### ✅ Answer: **B. Use exponential backoff and review the hot-key access pattern**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Một key quá phổ biến có thể tạo hot partition dù capacity tổng chưa dùng hết.
- Exponential backoff giảm áp lực retry; cần xem xét cách phân phối key hoặc dùng cache đọc như DAX khi phù hợp.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| A | Retry liên tục làm tăng tải lên partition đang nghẽn. |
| C | Item lớn hơn làm tăng capacity cần thiết, không khắc phục hot key. |
| D | Scan tạo thêm tải đọc và không giải quyết nguyên nhân phân phối request. |

### 📝 Vocabulary Analysis:
- **hot key**: khóa nhận lượng truy cập tập trung cao.
- **throttled**: request bị giới hạn do vượt capacity khả dụng.
- **exponential backoff**: tăng dần khoảng chờ giữa các lần retry.

### 🔄 Context Variations:
- **If throttling is caused by a GSI** → Check the GSI's partition key and provisioned capacity.
- **If the workload is read-heavy for a hot item** → Consider DAX or another suitable caching design.

---

## 7. Chọn LSI hay GSI

### Question 7:
A table must support queries using a **different partition key** from the base table. The index can have its own read and write capacity. Which index should be created?

```
A. A Local Secondary Index (LSI)
B. A Global Secondary Index (GSI)
C. A FilterExpression
D. A DynamoDB Stream
```

### ✅ Answer: **B. A Global Secondary Index (GSI)**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- GSI có thể dùng Primary Key khác với table chính và có RCU/WCU được provision riêng.
- GSI có thể được thêm sau khi tạo table.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| A | LSI dùng cùng Partition Key của table và phải được khai báo khi tạo table. |
| C | FilterExpression lọc kết quả, không tạo một access path/index mới. |
| D | Stream ghi lại thay đổi item, không phải index truy vấn. |

### 📝 Vocabulary Analysis:
- **base table**: table gốc chứa dữ liệu.
- **different partition key**: Partition Key khác với khóa của table gốc.
- **provision capacity**: cấp read/write capacity cho index.

### 🔄 Context Variations:
- **If the Partition Key stays the same but an alternate Sort Key is required at table creation** → Consider an LSI.
- **If a GSI is write-throttled** → Investigate its capacity and key distribution; writes to the base table can also be affected.

---

## 8. Ngăn cập nhật ghi đè

### Question 8:
Two clients may update the same DynamoDB item concurrently. The application must reject an update if the item changed after it was read. Which approach should it use?

```
A. Scan the table before every update
B. Use a version attribute with a conditional write
C. Use a larger item size
D. Enable TTL on the table
```

### ✅ Answer: **B. Use a version attribute with a conditional write**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Optimistic Locking lưu version attribute và chỉ cập nhật nếu version vẫn khớp với giá trị đã đọc.
- Nếu client khác đã cập nhật item, điều kiện không còn đúng và thao tác bị từ chối thay vì ghi đè thay đổi mới.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| A | Scan không bảo vệ tính đồng thời giữa thời điểm đọc và ghi. |
| C | Kích thước item không giải quyết xung đột cập nhật. |
| D | TTL xóa item theo thời hạn, không kiểm soát optimistic locking. |

### 📝 Vocabulary Analysis:
- **concurrently**: đồng thời.
- **version attribute**: attribute lưu phiên bản item.
- **reject an update**: từ chối cập nhật khi điều kiện không còn đúng.

### 🔄 Context Variations:
- **If a new item must not overwrite an existing item** → Use `attribute_not_exists` in a conditional write.
- **If several items must succeed or fail together** → Use a DynamoDB transaction.

---

## 9. Xử lý BatchWriteItem chưa hoàn tất

### Question 9:
A `BatchWriteItem` request returns some entries in `UnprocessedItems`. What should the application do?

```
A. Assume the entire batch succeeded
B. Retry only the unprocessed entries with exponential backoff
C. Retry all entries continuously without delay
D. Convert the request to a Scan
```

### ✅ Answer: **B. Retry only the unprocessed entries with exponential backoff**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Batch operations có thể thành công một phần; các thao tác chưa hoàn tất được trả trong `UnprocessedItems`.
- Retry riêng phần còn lại với exponential backoff tránh lặp lại thao tác đã thành công và giảm tải khi capacity bị giới hạn.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| A | Các entry trả về trong `UnprocessedItems` chưa được xử lý thành công. |
| C | Gửi lại toàn bộ batch có thể lặp thao tác và retry dồn dập làm tăng throttling. |
| D | Scan không xử lý các write chưa hoàn tất. |

### 📝 Vocabulary Analysis:
- **unprocessed entries**: các thao tác chưa được xử lý.
- **partial success**: chỉ một phần thao tác trong batch thành công.
- **retry**: thử lại thao tác thất bại/chưa hoàn tất.

### 🔄 Context Variations:
- **If an item must be updated rather than put or deleted in a batch** → Use `UpdateItem`; `BatchWriteItem` does not support updates.
- **If writes across items must be atomic** → Use `TransactWriteItems` instead of a batch write.

---

## 10. Dùng DynamoDB Transactions

### Question 10:
A purchase must update an inventory item and create an order item. Either both changes must be committed or neither may be committed. Which DynamoDB feature should be used?

```
A. Two independent PutItem requests
B. TransactWriteItems
C. BatchWriteItem
D. A Scan followed by DeleteItem
```

### ✅ Answer: **B. TransactWriteItems**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- `TransactWriteItems` phối hợp nhiều thao tác ghi theo kiểu all-or-nothing, giữ invariant giữa inventory và order.
- Transaction cung cấp tính chất ACID nhưng transactional read/write tiêu thụ khoảng gấp đôi capacity tiêu chuẩn theo slide.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| A | Hai request độc lập có thể chỉ một request thành công, tạo trạng thái không nhất quán. |
| C | BatchWriteItem không bảo đảm atomicity và không hỗ trợ cập nhật item. |
| D | Scan/Delete không thực hiện hai cập nhật nghiệp vụ nguyên tử. |

### 📝 Vocabulary Analysis:
- **committed**: được ghi nhận thành công.
- **all-or-nothing**: tất cả thao tác thành công hoặc không thao tác nào được áp dụng.
- **invariant**: điều kiện nghiệp vụ phải luôn được giữ đúng.

### 🔄 Context Variations:
- **If only one item needs a conditional update** → Use `UpdateItem` with a Condition Expression.
- **If a multi-item transaction is used frequently** → Include its higher RCU/WCU cost in capacity planning.

---

## 11. Giảm độ trễ đọc Hot Key

### Question 11:
An application performs frequent reads for a small set of DynamoDB items. The team wants a managed in-memory cache that works with DynamoDB APIs and reduces read latency. Which service is the best fit?

```
A. DynamoDB Accelerator (DAX)
B. AWS Data Pipeline
C. DynamoDB Streams
D. Point-in-time Recovery
```

### ✅ Answer: **A. DynamoDB Accelerator (DAX)**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- DAX là cache in-memory được quản lý và tương thích API DynamoDB; phù hợp giảm độ trễ read/query và xử lý Hot Key đọc.
- Cache có TTL nên cần chấp nhận dữ liệu có thể chưa phản ánh thay đổi mới nhất trong thời gian cache.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Data Pipeline là lựa chọn trong slide cho sao chép/migration dữ liệu, không phải cache đọc. |
| C | Streams ghi nhận thay đổi item-level để xử lý sự kiện, không cache read responses. |
| D | PITR hỗ trợ khôi phục dữ liệu, không tăng tốc truy vấn. |

### 📝 Vocabulary Analysis:
- **in-memory cache**: cache lưu trong bộ nhớ.
- **read latency**: độ trễ khi đọc dữ liệu.
- **hot item**: item có lượng truy cập đọc tập trung cao.

### 🔄 Context Variations:
- **If the application needs a general-purpose cache with custom data structures** → Consider ElastiCache.
- **If every read must reflect the latest write immediately** → Avoid relying on cached data without validating consistency requirements.

---

## 12. Đọc thay đổi bằng DynamoDB Streams

### Question 12:
A Lambda function must process DynamoDB item changes as they occur. Which configuration allows Lambda to read the stream records?

```
A. Configure a DynamoDB Event Source Mapping and grant the required permissions
B. Add a FilterExpression to every PutItem request
C. Use a Global Secondary Index as the event source
D. Configure a DynamoDB TTL attribute only
```

### ✅ Answer: **A. Configure a DynamoDB Event Source Mapping and grant the required permissions**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Lambda đọc DynamoDB Streams qua Event Source Mapping; function cần quyền phù hợp để đọc stream.
- Theo slide, Lambda được gọi đồng bộ khi xử lý records từ Event Source Mapping.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | FilterExpression điều khiển kết quả đọc, không phát sự kiện thay đổi item. |
| C | GSI là index truy vấn, không phải nguồn stream. |
| D | TTL hết hạn item và có thể tạo stream record khi item bị xóa; TTL không cấu hình Lambda đọc stream. |

### 📝 Vocabulary Analysis:
- **item changes**: các thay đổi create/update/delete trên item.
- **Event Source Mapping**: cơ chế Lambda poll/đọc records từ event source được hỗ trợ.
- **grant permissions**: cấp quyền IAM cần thiết.

### 🔄 Context Variations:
- **If the application needs only keys of changed items** → Choose `KEYS_ONLY` as the stream view type.
- **If it needs the item state before and after a change** → Choose `NEW_AND_OLD_IMAGES`.

---

## 13. Hiểu thời điểm xóa của TTL

### Question 13:
A DynamoDB item has a TTL timestamp that expired one minute ago, but a query still returns the item. What should the application expect?

```
A. TTL deletion is asynchronous, so the expired item may remain visible until DynamoDB deletes it
B. The table is corrupted and must be deleted
C. TTL immediately deletes every expired item at the exact timestamp
D. The item must be copied to a Global Table first
```

### ✅ Answer: **A. TTL deletion is asynchronous, so the expired item may remain visible until DynamoDB deletes it**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Slide nêu item hết hạn được xóa trong vòng 48 giờ; trước khi bị xóa, item vẫn có thể xuất hiện trong read/query/scan.
- Nếu ứng dụng không được trả về item đã hết hạn, cần lọc theo TTL timestamp thay vì chờ cleanup.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Item còn hiện diện trong thời gian cleanup không đồng nghĩa table bị lỗi. |
| C | TTL không đảm bảo xóa đúng tại thời điểm timestamp. |
| D | Global Tables không phải điều kiện để TTL xóa item. |

### 📝 Vocabulary Analysis:
- **TTL timestamp**: thời điểm hết hạn được lưu trong attribute dạng Number/Unix Epoch.
- **asynchronous**: diễn ra không đồng bộ, không hoàn tất ngay tại thời điểm bắt đầu.
- **visible**: vẫn xuất hiện trong kết quả truy vấn.

### 🔄 Context Variations:
- **If expired items must not be returned immediately** → Filter them using the expiry timestamp in the read path.
- **If expired-item deletions must trigger downstream processing** → Enable DynamoDB Streams and handle the delete records.

---

## 14. Chọn Global Tables

### Question 14:
An application must serve reads and writes from multiple AWS Regions, with DynamoDB data replicated across those Regions. Which feature should it use?

```
A. Local Secondary Indexes
B. Global Tables
C. DynamoDB Local
D. DynamoDB Accelerator
```

### ✅ Answer: **B. Global Tables**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Global Tables cung cấp replication đa Region và mô hình multi-active theo nội dung slide.
- Đây là tính năng phục vụ truy cập dữ liệu ở nhiều Region, không phải index hay môi trường local.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| A | LSI tạo access path thay thế trong cùng table, không replication giữa Region. |
| C | DynamoDB Local phục vụ phát triển/kiểm thử cục bộ. |
| D | DAX là cache đọc trong bộ nhớ, không sao chép dữ liệu đa Region. |

### 📝 Vocabulary Analysis:
- **replicated across Regions**: dữ liệu được sao chép giữa các Region.
- **multi-active**: nhiều Region có thể phục vụ workload đồng thời.
- **Global Tables**: tính năng DynamoDB cho replication đa Region.

### 🔄 Context Variations:
- **If the requirement is to recover a table to an earlier point in time** → Use Point-in-time Recovery.
- **If the requirement is to query by an alternate key in one table** → Create an LSI or GSI as appropriate.
