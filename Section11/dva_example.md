# Câu hỏi ôn tập DVA-C02 - Section 11: AWS Lambda

> *Câu hỏi và lựa chọn bằng tiếng Anh; phần giải thích bằng tiếng Việt.*

---

## 1. Xử lý ảnh theo sự kiện

### Question 1:
A company needs to create thumbnails when users upload images to S3. The processing is short-lived and should scale automatically without managing servers. Which service is the best fit?

```
A. Amazon EC2 with a fixed number of instances
B. AWS Lambda triggered by S3 events
C. Amazon RDS
D. AWS CloudFormation StackSets
```

### ✅ Answer: **B. AWS Lambda triggered by S3 events**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- S3 có thể gọi Lambda bất đồng bộ khi có object event; Lambda chỉ chạy khi có sự kiện và tự mở rộng theo lượng công việc.
- Đây là xử lý ngắn, theo sự kiện, không cần quản lý máy chủ chạy liên tục.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | EC2 cần quản lý instance và có thể chạy nhàn rỗi giữa các lần upload. |
| C | RDS lưu dữ liệu quan hệ, không thực thi xử lý ảnh theo event. |
| D | StackSets triển khai CloudFormation stacks trên nhiều account/Region. |

### 📝 Vocabulary Analysis:
- **thumbnail**: ảnh thu nhỏ được tạo từ ảnh gốc.
- **short-lived**: chạy trong thời gian ngắn.
- **triggered by**: được kích hoạt bởi một sự kiện.

### 🔄 Context Variations:
- **If the processing requires a continuously running service or arbitrary Docker workload** → Consider ECS/Fargate or EC2.
- **If the event must be consumed from an SQS queue** → Use an Event Source Mapping.

---

## 2. Gọi Lambda đồng bộ qua API

### Question 2:
An API Gateway request must wait for a Lambda function to calculate a result and return it in the same HTTP response. Which invocation type should be used?

```
A. Synchronous invocation
B. Asynchronous invocation
C. Event Source Mapping
D. CloudFront invalidation
```

### ✅ Answer: **A. Synchronous invocation**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Gọi đồng bộ trả kết quả ngay cho bên gọi; API Gateway có thể chờ Lambda hoàn tất để gửi HTTP response.
- Bên gọi chịu trách nhiệm xử lý lỗi/retry theo yêu cầu.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | Gọi bất đồng bộ không yêu cầu caller chờ kết quả function. |
| C | Event Source Mapping dùng để Lambda poll các nguồn như SQS/Kinesis/DynamoDB Streams. |
| D | Invalidation làm mới CloudFront cache, không quyết định cách Lambda được gọi. |

### 📝 Vocabulary Analysis:
- **same HTTP response**: cùng response HTTP đang chờ của request ban đầu.
- **synchronous invocation**: caller chờ function hoàn tất và nhận kết quả.
- **caller**: thành phần gọi function.

### 🔄 Context Variations:
- **If S3 should notify Lambda and does not need a response** → Use asynchronous invocation.
- **If records must be polled from SQS** → Use Event Source Mapping.

---

## 3. S3 event và Event Source Mapping

### Question 3:
An S3 object-created event should invoke a Lambda function, and S3 should not wait for the function result. Which invocation model should be used?

```
A. Synchronous invocation
B. Asynchronous invocation
C. Event Source Mapping for S3
D. Reserved concurrency
```

### ✅ Answer: **B. Asynchronous invocation**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Slide liệt kê S3 trong các nguồn asynchronous invocation. Event được chuyển vào hàng đợi để Lambda xử lý sau.
- Event Source Mapping áp dụng cho các nguồn cần được poll như SQS, Kinesis Data Streams và DynamoDB Streams, không phải S3 notification.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Đồng bộ yêu cầu bên gọi chờ response; đề bài không cần kết quả ngay. |
| C | S3 không nằm trong danh sách Event Source Mapping của slide. |
| D | Reserved concurrency giới hạn concurrency của function, không chọn loại invocation. |

### 📝 Vocabulary Analysis:
- **object-created event**: sự kiện phát sinh khi object được tạo trong S3.
- **invocation model**: cách dịch vụ gọi function và nhận/xử lý kết quả.
- **poll**: chủ động hỏi nguồn để lấy records.

### 🔄 Context Variations:
- **If Lambda must poll messages from SQS** → Use Event Source Mapping.
- **If an API client must receive the computed result immediately** → Use synchronous invocation.

---

## 4. Poll SQS bằng Event Source Mapping

### Question 4:
A Lambda function must process messages from an SQS queue. The function should not implement its own polling loop. Which feature should be configured?

```
A. S3 event notification
B. Lambda Event Source Mapping
C. CloudFront Function
D. Resource-based policy only
```

### ✅ Answer: **B. Lambda Event Source Mapping**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Event Source Mapping poll SQS, lấy records và gọi Lambda đồng bộ để xử lý.
- Lambda execution role cần quyền đọc nguồn SQS.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | S3 notification dùng cho sự kiện S3, không poll SQS. |
| C | CloudFront Function chạy tại edge để xử lý request/response, không đọc hàng đợi. |
| D | Resource-based policy cấp quyền cho bên gọi Lambda, không tạo polling từ SQS. |

### 📝 Vocabulary Analysis:
- **polling loop**: vòng lặp liên tục hỏi nguồn để lấy dữ liệu.
- **Event Source Mapping**: cấu hình Lambda đọc records từ một event source được hỗ trợ.
- **execution role**: IAM Role mà function dùng để truy cập dịch vụ AWS.

### 🔄 Context Variations:
- **If the source is DynamoDB Streams or Kinesis Data Streams** → Event Source Mapping is also supported.
- **If S3 invokes the function on object creation** → Configure S3 asynchronous invocation instead.

---

## 5. Retry và xử lý duplicate của asynchronous invocation

### Question 5:
An asynchronously invoked Lambda function sometimes fails after writing to a database. Retries can repeat the write. Which design best handles this behavior?

```
A. Make the operation idempotent and configure a failure destination or DLQ
B. Disable all logging
C. Increase the function timeout to 15 minutes
D. Use CloudFront Signed Cookies
```

### ✅ Answer: **A. Make the operation idempotent and configure a failure destination or DLQ**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Lambda retry asynchronous invocation khi function lỗi; thao tác có thể được gọi lại.
- Idempotency hạn chế side effect bị lặp; DLQ/Destination giữ kết quả thất bại để điều tra hoặc xử lý tiếp.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | Tắt logs làm giảm khả năng điều tra, không ngăn retries hoặc duplicate writes. |
| C | Timeout dài hơn không ngăn việc gọi lại thao tác đã chạy. |
| D | Signed Cookies kiểm soát truy cập nội dung CloudFront, không xử lý retry Lambda. |

### 📝 Vocabulary Analysis:
- **idempotent**: gọi lặp cùng thao tác vẫn không tạo thêm tác động ngoài ý muốn.
- **retry**: thử lại sau khi lần xử lý trước thất bại.
- **failure destination**: đích nhận kết quả invocation thất bại.

### 🔄 Context Variations:
- **If the invocation succeeds but its result must be routed elsewhere** → Configure an asynchronous success destination.
- **If SQS Event Source Mapping discards a failed batch** → Configure an SQS/SNS destination or DLQ for the discarded batch.

---

## 6. Cấp quyền cho Lambda truy cập DynamoDB

### Question 6:
A Lambda function must write items to a DynamoDB table and send execution logs to CloudWatch Logs. Which permissions should be granted to the function?

```
A. Add the permissions to the Lambda execution role
B. Add a resource-based policy to the DynamoDB table only
C. Add the permissions to the API Gateway role
D. Make the DynamoDB table publicly accessible
```

### ✅ Answer: **A. Add the permissions to the Lambda execution role**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Execution Role là danh tính mà mã Lambda dùng khi gọi dịch vụ AWS.
- Role cần quyền ghi DynamoDB và policy ghi logs CloudWatch, chẳng hạn policy cơ bản được quản lý cho Lambda logs.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | Resource-based policy trên bảng không thay thế quyền identity của function để gọi API. |
| C | API Gateway role không phải danh tính runtime của mã Lambda. |
| D | Không cần mở DynamoDB công khai; cần cấp IAM permissions phù hợp. |

### 📝 Vocabulary Analysis:
- **execution role**: IAM Role được Lambda assume khi function chạy.
- **grant permissions**: cấp quyền thực hiện các API actions.
- **execution logs**: nhật ký do function ghi trong lúc chạy.

### 🔄 Context Variations:
- **If S3 must invoke the Lambda function** → Add a resource-based policy to the function for S3.
- **If the function reads from SQS through Event Source Mapping** → Grant the execution role permission to read the queue.

---

## 7. Cho phép S3 gọi Lambda

### Question 7:
An S3 bucket must invoke a Lambda function when a new object is created. Which permission mechanism allows the S3 service to invoke the function?

```
A. Lambda execution role only
B. A resource-based policy on the Lambda function
C. A CloudWatch metric filter
D. A CloudFront Cache Policy
```

### ✅ Answer: **B. A resource-based policy on the Lambda function**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Resource-based policy trên Lambda cấp quyền cho dịch vụ S3 được invoke function.
- Execution Role cấp quyền cho mã function truy cập các dịch vụ khác, không cấp quyền cho S3 gọi function.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Execution Role là quyền runtime của Lambda, không phải quyền invoke của dịch vụ S3. |
| C | Metric filter tạo metric từ logs, không cấp quyền invoke. |
| D | Cache Policy điều khiển Cache Key của CloudFront, không liên quan quyền Lambda. |

### 📝 Vocabulary Analysis:
- **resource-based policy**: policy gắn trên tài nguyên để xác định principal nào được truy cập.
- **invoke**: gọi function để thực thi.
- **principal**: người dùng, role hoặc dịch vụ được xét quyền.

### 🔄 Context Variations:
- **If Lambda itself must call DynamoDB** → Grant the action to its execution role.
- **If API Gateway invokes Lambda** → Configure the appropriate invoke permission for the API Gateway service.

---

## 8. Truy cập RDS trong VPC

### Question 8:
A Lambda function must connect to an RDS database in a private subnet. What must be configured for the function to reach the database?

```
A. VPC ID, subnets, and Security Groups
B. A public IP for the Lambda function
C. CloudFront VPC Origins
D. A Lambda Layer containing the database password
```

### ✅ Answer: **A. VPC ID, subnets, and Security Groups**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Lambda mặc định chạy ngoài VPC của người dùng. Cấu hình VPC, subnets và Security Groups để Lambda tạo ENI trong subnet có đường đến RDS.
- Security Groups phải cho phép kết nối giữa Lambda và database theo port cần thiết.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | Lambda không nhận public IP chỉ vì được gắn vào VPC. |
| C | VPC Origins dành cho CloudFront truy cập origin private, không kết nối Lambda đến RDS. |
| D | Layer đóng gói dependencies, không cấu hình đường mạng hay secret an toàn. |

### 📝 Vocabulary Analysis:
- **private subnet**: subnet không định tuyến trực tiếp đến Internet Gateway.
- **ENI**: Elastic Network Interface được Lambda tạo trong subnet.
- **Security Group**: firewall stateful kiểm soát traffic đến/đi.

### 🔄 Context Variations:
- **If the function also needs public Internet access** → Route outbound traffic through a NAT Gateway/Instance.
- **If it only needs private access to an AWS service** → Consider a VPC Endpoint where supported.

---

## 9. Internet access từ Lambda trong VPC

### Question 9:
A Lambda function needs outbound Internet access while accessing resources in a VPC. Which network configuration should be used?

```
A. Place the function in a public subnet; it automatically receives a public IP
B. Place the function in a private subnet with a route through a NAT Gateway or NAT Instance
C. Attach an Internet Gateway directly to the function's network interface
D. A CloudWatch Logs subscription
```

### ✅ Answer: **B. Place the function in a private subnet with a route through a NAT Gateway or NAT Instance**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Lambda trong VPC không được cấp public IP chỉ vì đặt trong public subnet.
- Để đi ra Internet, function trong private subnet cần route qua NAT Gateway/Instance theo kiến trúc trong slide.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Internet Gateway không tự cấp public IP hoặc đường Internet cho ENI Lambda. |
| C | Lambda trong VPC không nhận public IP theo cấu hình subnet. |
| D | Subscription chuyển log events, không tạo kết nối mạng outbound. |

### 📝 Vocabulary Analysis:
- **outbound Internet access**: kết nối từ workload trong VPC ra Internet.
- **NAT Gateway**: cho phép tài nguyên trong private subnet khởi tạo kết nối ra ngoài.
- **route**: quy tắc định tuyến traffic trong subnet.

### 🔄 Context Variations:
- **If the function only needs S3 access without public Internet** → Use an S3 VPC Endpoint.
- **If no outbound Internet is required** → Do not add NAT solely for Lambda placement.

---

## 10. Cấp RAM cho function bị giới hạn CPU

### Question 10:
A Lambda function is CPU-bound and takes too long to process each request. Which configuration change can provide more CPU resources?

```
A. Increase the function's memory allocation and benchmark the result
B. Increase the CloudWatch Logs retention period
C. Add an S3 event notification
D. Enable a CloudFront Signed URL
```

### ✅ Answer: **A. Increase the function's memory allocation and benchmark the result**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Lambda cấp thêm CPU/network khi tăng RAM; với workload CPU-bound, tăng RAM có thể rút ngắn thời gian xử lý.
- Cần benchmark vì tăng RAM cũng ảnh hưởng chi phí và không phải mọi workload đều bị giới hạn CPU.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | Retention chỉ điều khiển thời gian lưu logs. |
| C | S3 notification là nguồn kích hoạt, không tăng tài nguyên CPU cho function. |
| D | Signed URL kiểm soát truy cập CloudFront, không thay đổi cấu hình Lambda. |

### 📝 Vocabulary Analysis:
- **CPU-bound**: workload bị giới hạn bởi năng lực tính toán CPU.
- **memory allocation**: dung lượng RAM cấp cho function.
- **benchmark**: đo hiệu năng trong điều kiện thử nghiệm có kiểm soát.

### 🔄 Context Variations:
- **If the function exceeds its maximum runtime** → Review the workload; Lambda timeout is limited to 15 minutes in the slide.
- **If the function needs more vCPU than one** → The slide describes increased CPU above 1,792 MB and code must use multithreading to benefit.

---

## 11. Reserved Concurrency và throttling

### Question 11:
A critical Lambda function is throttled when other functions consume the account's concurrency. The team wants to reserve a concurrency allocation for this function and set its maximum. Which feature should it configure?

```
A. Reserved concurrency
B. Provisioned Concurrency only
C. A longer timeout
D. CloudTrail Insights
```

### ✅ Answer: **A. Reserved concurrency**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Reserved concurrency dành phần concurrency cho function và đồng thời đặt giới hạn tối đa của function.
- Nó cô lập capacity khỏi các function khác; invocation vượt giới hạn reserved vẫn bị throttle.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | Provisioned Concurrency khởi tạo sẵn execution environments để giảm cold start, không thay cho việc đặt giới hạn concurrency dành riêng. |
| C | Timeout điều khiển thời gian chạy tối đa, không dành concurrency. |
| D | CloudTrail Insights phát hiện hoạt động API bất thường. |

### 📝 Vocabulary Analysis:
- **reserved concurrency**: phần concurrency được dành và giới hạn cho một function.
- **throttled**: invocation bị từ chối/giới hạn do vượt concurrency khả dụng.
- **allocation**: phần tài nguyên được phân bổ.

### 🔄 Context Variations:
- **If the function exceeds its reserved limit and needs more capacity** → Increase the appropriate concurrency quota/reservation.
- **If first-request latency from initialization is the issue** → Consider Provisioned Concurrency.

---

## 12. Loại bỏ cold start

### Question 12:
A latency-sensitive Lambda function must have execution environments initialized before requests arrive. Which feature should be used?

```
A. Reserved concurrency
B. Provisioned Concurrency
C. A larger timeout
D. An SQS dead-letter queue
```

### ✅ Answer: **B. Provisioned Concurrency**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Provisioned Concurrency cấp concurrency trước invocation, nhờ đó môi trường sẵn sàng và giảm cold start.
- Có thể dùng Application Auto Scaling để điều chỉnh concurrency theo lịch hoặc mức utilization.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Reserved concurrency đặt mức dành riêng/giới hạn nhưng không khởi tạo trước execution environments. |
| C | Timeout chỉ cho phép function chạy lâu hơn, không giảm thời gian khởi tạo. |
| D | DLQ lưu invocation lỗi, không giữ sẵn môi trường chạy. |

### 📝 Vocabulary Analysis:
- **cold start**: độ trễ khởi tạo môi trường khi Lambda cần instance mới.
- **Provisioned Concurrency**: số execution environments được chuẩn bị trước.
- **latency-sensitive**: yêu cầu nghiêm ngặt về độ trễ.

### 🔄 Context Variations:
- **If only other functions must not consume this function's allocation** → Use Reserved Concurrency.
- **If invocations arrive on a predictable schedule** → Application Auto Scaling can schedule Provisioned Concurrency.

---

## 13. Thao tác ở CloudFront Edge

### Question 13:
A team needs to normalize query strings and modify HTTP headers at the viewer-request stage with very low startup latency. Which edge option should it choose?

```
A. CloudFront Functions
B. Lambda@Edge
C. AWS Lambda in a private subnet
D. Amazon EC2 Auto Scaling
```

### ✅ Answer: **A. CloudFront Functions**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- CloudFront Functions phù hợp với thao tác nhẹ tại Viewer Request/Response như chuẩn hóa Cache Key, sửa header và URL rewrite; slide nêu startup dưới 1 ms.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | Lambda@Edge phù hợp khi cần thời gian xử lý dài hơn, thư viện, network hoặc truy cập body/filesystem. |
| C | Lambda trong VPC chạy tại Region/VPC, không phải lựa chọn edge nhẹ nhất cho viewer request. |
| D | EC2 Auto Scaling quản lý compute instance, không thực thi logic trực tiếp tại CloudFront edge. |

### 📝 Vocabulary Analysis:
- **normalize query strings**: chuẩn hóa query parameters trước khi dùng làm Cache Key.
- **viewer request**: request từ người xem ngay sau khi CloudFront nhận được.
- **edge option**: cơ chế chạy mã tại vị trí biên gần người dùng.

### 🔄 Context Variations:
- **If the code needs network access or a third-party library** → Consider Lambda@Edge.
- **If only a simple header or URL rewrite is required** → CloudFront Functions is usually the lighter option.
