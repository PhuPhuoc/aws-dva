# Câu hỏi ôn tập DVA-C02 - Section 7: CloudFront và Elastic Beanstalk

> *Câu hỏi và lựa chọn bằng tiếng Anh; phần giải thích bằng tiếng Việt.*

---

## 1. Phân phối nội dung toàn cầu

### Question 1:
A company serves static images to users around the world. It wants to reduce latency by caching content close to users. Which service is the best fit?

```
A. Amazon CloudFront
B. S3 Cross-Region Replication only
C. AWS Elastic Beanstalk
D. Amazon RDS
```

### ✅ Answer: **A. Amazon CloudFront**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- CloudFront cache nội dung tại Edge Locations, giảm số lần phải đọc từ origin và cải thiện độ trễ cho người dùng ở xa.
- S3 Cross-Region Replication tạo bản sao theo Region; nó không thay thế mạng Edge cache toàn cầu.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | Replication phù hợp tạo bản sao ở các Region đã cấu hình, không phân phối trực tiếp qua edge toàn cầu. |
| C | Elastic Beanstalk triển khai ứng dụng, không phải CDN. |
| D | RDS là cơ sở dữ liệu, không cache tệp tại edge. |

### 📝 Vocabulary Analysis:
- **static images**: ảnh tĩnh ít thay đổi.
- **cache close to users**: lưu bản sao tại vị trí mạng gần người dùng.
- **origin**: nơi CloudFront lấy nội dung gốc.

### 🔄 Context Variations:
- **If dynamic data must be replicated to a few AWS Regions for local reads** → Consider S3 Cross-Region Replication.
- **If the requirement is to run application code on managed AWS infrastructure** → Consider Elastic Beanstalk.

---

## 2. Giữ riêng tư S3 origin

### Question 2:
A company uses an S3 bucket as a CloudFront origin. The bucket must remain private, and only the CloudFront distribution may read its objects. Which configuration provides this access while blocking public access?

```
A. Enable public read access on the bucket
B. Configure an Origin Access Control (OAC), restrict the bucket policy to the distribution, and block public access
C. Create an S3 Cross-Region Replication rule
D. Put the bucket in a public subnet
```

### ✅ Answer: **B. Configure an Origin Access Control (OAC), restrict the bucket policy to the distribution, and block public access**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- OAC ký các request từ CloudFront đến S3 REST origin; bucket policy cần cho phép `s3:GetObject` đối với `cloudfront.amazonaws.com` chỉ khi `AWS:SourceArn` khớp với distribution này.
- Bật S3 Block Public Access để người dùng không thể truy cập vòng qua CloudFront. OAC không hỗ trợ S3 website endpoint.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Public read cho phép truy cập bucket ngoài CloudFront. |
| C | Replication tạo bản sao ở Region khác, không giới hạn quyền truy cập origin. |
| D | S3 bucket không được đặt trong subnet như EC2 instance. |

### 📝 Vocabulary Analysis:
- **private bucket**: bucket không cho truy cập công khai.
- **origin access**: quyền CloudFront dùng để đọc nội dung từ origin.
- **distribution**: cấu hình CloudFront phục vụ nội dung.

### 🔄 Context Variations:
- **If the origin is an application in a private VPC subnet** → Consider CloudFront VPC Origins for supported origins.
- **If clients should access S3 directly** → Configure a separate access policy for that requirement rather than making the bucket public by default.

---

## 3. Cache theo query string và header

### Question 3:
A website returns different content based on a language query string and an HTTP header. CloudFront currently serves the same cached response for both languages. What should be changed?

```
A. Add the relevant query string and header to the Cache Policy
B. Disable all caching for the distribution
C. Add an origin group with a secondary S3 bucket
D. Enable Elastic Beanstalk rolling deployments
```

### ✅ Answer: **A. Add the relevant query string and header to the Cache Policy**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Mặc định Cache Key gồm hostname và đường dẫn; query string/header chỉ phân biệt biến thể nếu được đưa vào Cache Key qua Cache Policy.
- Cấu hình đúng các thuộc tính làm thay đổi nội dung giúp CloudFront trả đúng phiên bản cho từng ngôn ngữ.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | Tắt toàn bộ cache làm tăng tải origin, trong khi chỉ cần phân biệt các biến thể liên quan. |
| C | Origin group dùng cho failover, không tạo Cache Key theo ngôn ngữ. |
| D | Deployment mode của Elastic Beanstalk không điều khiển CloudFront Cache Key. |

### 📝 Vocabulary Analysis:
- **Cache Key**: khóa nhận diện từng đối tượng được lưu trong cache.
- **query string**: phần tham số sau dấu `?` trong URL.
- **HTTP header**: trường metadata trong request/response HTTP.

### 🔄 Context Variations:
- **If a cookie changes the response body** → Include that cookie in the Cache Policy.
- **If a header does not affect the response** → Do not add it to the Cache Key, to avoid unnecessary cache variants.

---

## 4. Tự động chuyển origin khi lỗi

### Question 4:
A CloudFront distribution must serve content from a secondary origin when its primary origin fails. Which feature should be configured?

```
A. Cache invalidation
B. Origin Group with a primary and secondary origin
C. Signed Cookies
D. CloudFront Price Class 100
```

### ✅ Answer: **B. Origin Group with a primary and secondary origin**

### 📖 Explanation (Vietnamese):

- Origin Group có primary và secondary origin; CloudFront chỉ failover theo mã phản hồi lỗi được cấu hình cho group, không phải mọi lỗi bất kỳ. Cần chọn các mã phù hợp với hành vi origin.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Invalidation xóa đối tượng cache, không cấu hình failover giữa origins. |
| C | Signed Cookies kiểm soát quyền truy cập nhiều tệp, không chuyển origin. |
| D | Price Class chọn phạm vi Edge Locations theo chi phí, không cấu hình failover. |

### 📝 Vocabulary Analysis:
- **primary origin**: origin được ưu tiên phục vụ trước.
- **secondary origin**: origin dự phòng dùng khi origin chính lỗi.
- **failover**: chuyển sang thành phần dự phòng khi thành phần chính thất bại.

### 🔄 Context Variations:
- **If different URL paths should use different origins** → Configure Cache Behaviors.
- **If the source content changed and only cache needs refreshing** → Create an invalidation.

## 4A. Điều kiện kích hoạt Origin failover

### Question 4A:
A CloudFront distribution receives a **GET** viewer request for an object. The primary origin returns an HTTP status code configured for origin failover. Which response should CloudFront use?

```
A. The response from the secondary origin
B. Any change to the S3 object version
C. The viewer's country
D. The distribution's Price Class
```

### ✅ Answer: **A. The response from the secondary origin**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Origin Group failover dựa trên các status code được cấu hình; không nên hiểu rằng mọi lỗi hoặc mọi lần cập nhật origin đều tự động kích hoạt chuyển đổi.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | Thay đổi object không tự cấu hình hoặc kích hoạt điều kiện failover. |
| C | Geo Restriction quyết định quốc gia được phục vụ, không phải điều kiện chuyển origin. |
| D | Price Class điều chỉnh phạm vi Edge Locations, không định nghĩa lỗi origin. |

### 📝 Vocabulary Analysis:
- **failover status codes**: mã trạng thái HTTP đã chọn làm điều kiện chuyển sang origin phụ.
- **origin group**: nhóm origin chính và origin dự phòng.

### 🔄 Context Variations:
- **If the primary returns a configured failover status code** → CloudFront can request the content from the secondary origin.
- **If a status code is not configured for failover** → Do not assume CloudFront will switch origins for it.

---


## 5. Làm mới nội dung đang được cache

### Question 5:
A team updates files at its CloudFront origin but users continue to receive the old cached files. The TTL has not expired. What should the team do to refresh only the changed path?

```
A. Create a CloudFront invalidation for the changed path
B. Change the origin group to a secondary origin
C. Enable detailed monitoring
D. Clone the Elastic Beanstalk environment
```

### ✅ Answer: **A. Create a CloudFront invalidation for the changed path**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- CloudFront không tự biết origin đã đổi; nội dung cache vẫn được phục vụ đến khi TTL hết hạn.
- Invalidation yêu cầu CloudFront làm mới path được chỉ định mà không cần chờ TTL.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | Origin Group dùng cho failover, không xóa bản cache cũ. |
| C | Detailed monitoring theo dõi metric EC2, không tác động đến CloudFront cache. |
| D | Clone environment tạo bản sao cấu hình Elastic Beanstalk, không làm mới cache. |

### 📝 Vocabulary Analysis:
- **TTL**: thời gian một đối tượng có thể nằm trong cache trước khi hết hạn.
- **invalidation**: yêu cầu xóa đối tượng cache để CloudFront lấy nội dung mới.
- **changed path**: đường dẫn của nội dung vừa được cập nhật.

### 🔄 Context Variations:
- **If the update can wait until the current cache expires** → Let the TTL expire.
- **If every cached object must be refreshed** → Invalidate `/*`, understanding that this removes all matching cache entries.

---

## 6. Cấp quyền cho nhiều tệp riêng tư

### Question 6:
A media company sells access to several private files in a CloudFront distribution. A customer should access multiple files using one time-limited authorization. Which feature is appropriate?

```
A. A separate Signed URL for every file
B. A CloudFront Signed Cookie
C. A CloudFront invalidation
D. A Cache Behavior with a default path only
```

### ✅ Answer: **B. A CloudFront Signed Cookie**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Signed Cookie có thể cấp quyền truy cập nhiều tệp bằng một cookie có policy và thời hạn.
- Signed URL phù hợp khi cấp quyền cho từng tệp riêng.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Mỗi Signed URL cấp quyền cho một tệp, không phù hợp khi muốn một cơ chế cho nhiều tệp. |
| C | Invalidation làm mới cache, không cấp quyền truy cập. |
| D | Default behavior định tuyến request, không xác thực người dùng. |

### 📝 Vocabulary Analysis:
- **time-limited authorization**: quyền truy cập chỉ có hiệu lực trong thời gian được đặt.
- **Signed Cookie**: cookie ký số cấp quyền cho nhiều tài nguyên CloudFront.
- **premium content**: nội dung trả phí hoặc chỉ dành cho người dùng được cấp quyền.

### 🔄 Context Variations:
- **If access is needed for exactly one file** → Use a Signed URL.
- **If access should be restricted to an IP range** → Include the allowed IP range in the signed policy.

---

## 7. Chọn deployment mode cho production

### Question 7:
A production application needs a deployment with **no downtime** and a **fast rollback** if the new version is unhealthy. The team can temporarily run double capacity. Which Elastic Beanstalk deployment option is the best fit?

```
A. All at once
B. Rolling
C. Immutable
D. Rolling with additional batches
```

### ✅ Answer: **C. Immutable**

### 📖 Explanation (Vietnamese):

**Tại sao chọn C:**
- Immutable tạo instance mới trong ASG tạm, chờ health tốt rồi mới chuyển toàn bộ.
- Nếu thất bại, có thể rollback nhanh bằng cách terminate ASG mới; đổi lại cần capacity và chi phí gần gấp đôi trong thời gian triển khai.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | All at once nhanh nhất nhưng có thể gây downtime. |
| B | Rolling cập nhật theo batch và có thể chạy dưới capacity; rollback không đơn giản như hủy ASG tạm. |
| D | Rolling with additional batches giữ capacity nhưng không có cơ chế cô lập version trong ASG tạm như Immutable. |

### 📝 Vocabulary Analysis:
- **immutable deployment**: triển khai version mới lên instance mới thay vì ghi đè instance cũ.
- **rollback**: quay lại version trước khi deployment mới thất bại.
- **health check**: kiểm tra instance/ứng dụng có hoạt động bình thường không.

### 🔄 Context Variations:
- **If the team wants to send only a small percentage of traffic to the new version** → Use Traffic Splitting.
- **If the team wants two separate environments and a later URL cutover** → Use a Blue/Green approach.

---

## 8. Blue/Green với Elastic Beanstalk

### Question 8:
A team wants to validate a new application version in a separate Elastic Beanstalk environment, then switch traffic after approval. Which approach should it use?

```
A. All at once deployment
B. Blue/Green using a new environment and a URL or Route 53 switch
C. Change the load balancer type in the existing environment
D. Delete the existing environment before deploying the new version
```

### ✅ Answer: **B. Blue/Green using a new environment and a URL or Route 53 switch**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Blue/Green không phải deployment mode trực tiếp của Elastic Beanstalk; tạo environment mới, kiểm tra độc lập rồi chuyển traffic bằng swap URL hoặc Route 53.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | All at once cập nhật ngay trong environment hiện tại, không tạo môi trường kiểm thử riêng. |
| C | Không thể đổi loại Load Balancer sau khi tạo environment; chỉ đổi cấu hình được. |
| D | Xóa environment trước khi kiểm tra version mới làm mất khả năng chuyển traffic an toàn. |

### 📝 Vocabulary Analysis:
- **Blue/Green**: chạy hai environments riêng để kiểm tra version mới trước khi chuyển traffic.
- **cutover**: thời điểm chuyển người dùng sang environment mới.
- **swap URL**: đổi URL giữa hai environments Elastic Beanstalk.

### 🔄 Context Variations:
- **If only a small portion of users should test the new version** → Use Traffic Splitting/Canary deployment.
- **If the environment is production and rollback must be fast** → Immutable is another option with temporary extra capacity.

---

## 9. Tách RDS production khỏi environment

### Question 9:
A production application uses an RDS database created as part of an Elastic Beanstalk environment. The team wants to terminate and recreate application environments without tying the database lifecycle to them. What should it do?

```
A. Keep RDS inside the Elastic Beanstalk environment and clone it
B. Create RDS separately and configure the application with its connection string
C. Store database files in the Elastic Beanstalk application version
D. Use a CloudFront Origin Group for the database
```

### ✅ Answer: **B. Create RDS separately and configure the application with its connection string**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Slide khuyến nghị production RDS được tạo riêng; environment mới có thể trỏ đến database qua connection string.
- RDS tạo cùng Beanstalk phù hợp hơn với dev/test vì vòng đời database có thể gắn với environment.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Clone environment không sao chép dữ liệu RDS, và database vẫn gắn với vòng đời environment. |
| C | Application version không phải nơi lưu trữ database bền vững. |
| D | CloudFront Origin Group dùng cho failover giữa web origins, không quản lý RDS lifecycle. |

### 📝 Vocabulary Analysis:
- **database lifecycle**: vòng đời tạo, vận hành và xóa cơ sở dữ liệu.
- **connection string**: thông tin ứng dụng dùng để kết nối cơ sở dữ liệu.
- **decouple**: tách vòng đời một thành phần khỏi thành phần khác.

### 🔄 Context Variations:
- **If the environment is disposable dev/test and the database is also disposable** → An RDS instance inside the environment can be convenient.
- **If an existing production database must be retained during migration** → Snapshot, enable deletion protection, create a new environment pointing to it, then switch traffic.

---

## 10. Thay đổi loại Load Balancer

### Question 10:
An Elastic Beanstalk environment currently uses one load balancer type. The team must migrate to a different load balancer type. What is the supported migration approach in the slides?

```
A. Change the load balancer type in place
B. Create a new environment with the desired type, deploy the application, then switch the CNAME or Route 53 record
C. Clone the environment and edit only the load balancer type
D. Add a CloudFront invalidation
```

### ✅ Answer: **B. Create a new environment with the desired type, deploy the application, then switch the CNAME or Route 53 record**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Sau khi environment được tạo, không thể đổi loại Load Balancer; cần tạo environment mới, deploy ứng dụng rồi chuyển DNS/CNAME.
- Slide nêu không thể clone để thay loại Load Balancer.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Chỉ cấu hình Load Balancer thay đổi được, không đổi loại. |
| C | Quy trình migration trong slide yêu cầu environment mới, không clone environment cũ. |
| D | Invalidation chỉ làm mới CloudFront cache, không đổi hạ tầng Elastic Beanstalk. |

### 📝 Vocabulary Analysis:
- **load balancer type**: loại dịch vụ cân bằng tải được environment sử dụng.
- **CNAME swap**: chuyển tên miền từ environment cũ sang environment mới.
- **migration**: chuyển ứng dụng sang hạ tầng mới.

### 🔄 Context Variations:
- **If only load balancer settings need adjustment but the type stays the same** → Update the configuration of the existing environment.
- **If the application version changes but the load balancer type stays the same** → Choose an Elastic Beanstalk deployment mode.

---

## 11. Giới hạn nội dung theo quốc gia

### Question 11:
A company has content distribution rights only in the United States and Canada. It wants its CloudFront distribution to **block viewers from all other countries**. Which feature should it configure?

```
A. Geo Restriction with an allowlist for the United States and Canada
B. Signed Cookies
C. Field-Level Encryption
D. Price Class 100
```

### ✅ Answer: **A. Geo Restriction with an allowlist for the United States and Canada**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Geo Restriction cho phép chặn hoặc chỉ cho phép các quốc gia trong danh sách; quốc gia được xác định bằng cơ sở dữ liệu Geo-IP.
- Allowlist phù hợp khi chỉ Hoa Kỳ và Canada được phép truy cập.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | Signed Cookies kiểm soát quyền truy cập nội dung theo policy, không lọc quốc gia. |
| C | Field-Level Encryption mã hóa các trường nhạy cảm trong POST request. |
| D | Price Class chọn phạm vi Edge Locations theo chi phí, không giới hạn quốc gia được xem nội dung. |

### 📝 Vocabulary Analysis:
- **allowlist**: danh sách quốc gia được phép truy cập.
- **Geo-IP database**: cơ sở dữ liệu suy ra vị trí quốc gia từ địa chỉ IP.

### 🔄 Context Variations:
- **If access must be granted to a specific user or paid customer** → Use Signed URLs or Signed Cookies rather than Geo Restriction.

---

## 12. Mã hóa một số trường trong POST request

### Question 12:
A financial application sends POST requests through CloudFront. It must **encrypt up to 10 selected fields** near the viewer so that intermediaries cannot read those fields before the origin decrypts them. Which capability should be configured?

```
A. Origin Access Control
B. Field-Level Encryption with a public key and the selected fields
C. A CloudFront Signed URL
D. A Cache Policy that includes every request header
```

### ✅ Answer: **B. Field-Level Encryption with a public key and the selected fields**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Field-Level Encryption mã hóa các trường được chọn trong POST request ở edge gần người dùng bằng public key; origin dùng private key để giải mã.
- Slide cho phép chọn tối đa 10 trường. HTTPS vẫn cần để bảo vệ đường truyền.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | OAC kiểm soát quyền CloudFront truy cập S3 origin, không mã hóa trường POST. |
| C | Signed URL kiểm soát quyền truy cập một tệp, không mã hóa payload. |
| D | Cache Policy chọn dữ liệu tham gia Cache Key; nó không mã hóa dữ liệu. |

### 📝 Vocabulary Analysis:
- **Field-Level Encryption**: mã hóa một số trường cụ thể trong request.
- **asymmetric encryption**: mã hóa bằng public key và giải mã bằng private key.

---

## 13. Gửi CloudFront real-time logs

### Question 13:
An operations team wants CloudFront request records sent to **Kinesis Data Streams within seconds**. It must choose the sampling rate, logged fields, and cache behaviors to include. What should the team enable?

```
A. CloudFront standard logging to Amazon S3
B. CloudFront real-time logs
C. CloudTrail management events
D. VPC Flow Logs
```

### ✅ Answer: **B. CloudFront real-time logs**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Real-time logs gửi request records đến Kinesis Data Streams trong vài giây và cho phép chọn sampling rate, fields, cùng cache behaviors.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Standard logs được gửi đến S3, không phải luồng real-time tùy chọn của Kinesis. |
| C | CloudTrail ghi nhận hoạt động API trong AWS, không thay thế request logs của CloudFront. |
| D | VPC Flow Logs ghi nhận metadata lưu lượng IP của network interface, không ghi nội dung request CloudFront. |

### 📝 Vocabulary Analysis:
- **sampling rate**: tỷ lệ request được chọn để ghi log.
- **real-time logs**: log được gửi đến nơi nhận trong thời gian ngắn sau request.

---

## 14. Chọn CloudFront Price Class

### Question 14:
A company wants to reduce CloudFront data-transfer cost and accepts serving content from **only the least expensive edge locations**. Which price class should it choose?

```
A. Price Class All
B. Price Class 200
C. Price Class 100
D. Real-time logging
```

### ✅ Answer: **C. Price Class 100**

### 📖 Explanation (Vietnamese):

**Tại sao chọn C:**
- Theo slide, Price Class 100 chỉ dùng các khu vực có chi phí thấp nhất; đổi lại phạm vi edge hẹp hơn và hiệu năng có thể thấp hơn ở một số vị trí.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Price Class All dùng mọi khu vực để có phạm vi phân phối rộng nhất. |
| B | Price Class 200 bao phủ phần lớn khu vực và loại một số khu vực đắt nhất, không giới hạn ở nhóm rẻ nhất. |
| D | Real-time logging phục vụ quan sát request, không chọn phạm vi Edge Locations. |

### 📝 Vocabulary Analysis:
- **Price Class**: lựa chọn tập hợp Edge Locations dùng để cân bằng chi phí và phạm vi phân phối.

---

## 15. Deploy ứng dụng bằng EB CLI

### Question 15:
An automated pipeline must deploy a new application version to an existing Elastic Beanstalk environment. Which EB CLI command should it run?

```
A. `eb deploy`
B. `eb status`
C. `eb terminate`
D. `eb open`
```

### ✅ Answer: **A. `eb deploy`**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- `eb deploy` triển khai application version lên environment hiện tại và phù hợp đưa vào deployment pipeline.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | `eb status` xem trạng thái environment, không triển khai version. |
| C | `eb terminate` chấm dứt environment. |
| D | `eb open` mở ứng dụng trong trình duyệt. |

### 📝 Vocabulary Analysis:
- **deployment pipeline**: chuỗi bước tự động kiểm thử và triển khai ứng dụng.
- **application version**: phiên bản mã ứng dụng được lưu trong Elastic Beanstalk.

---

## 16. Chọn Elastic Beanstalk environment tier

### Question 16:
An application needs a separate environment tier for **background worker processes**, rather than serving incoming web requests. Which Elastic Beanstalk tier should it use?

```
A. Web Server tier
B. Worker tier
C. CloudFront VPC Origin
D. Immutable deployment
```

### ✅ Answer: **B. Worker tier**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Worker tier dành cho xử lý công việc nền; Web Server tier phục vụ ứng dụng web nhận request từ client.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| A | Web Server tier phục vụ request web, không phải tier dành riêng cho worker processes. |
| C | VPC Origin kết nối CloudFront với origin riêng tư, không phải Elastic Beanstalk tier. |
| D | Immutable là phương thức deploy version mới, không phải environment tier. |

### 📝 Vocabulary Analysis:
- **background worker**: tiến trình xử lý tác vụ nền thay vì trả response trực tiếp cho người dùng.
- **environment tier**: loại environment Elastic Beanstalk dùng cho một kiểu workload.

---

## 17. Cấu hình Elastic Beanstalk bằng `.ebextensions`

### Question 17:
A team needs to define `option_settings` and add supporting AWS resources in the Elastic Beanstalk application source bundle. Which approach meets this requirement?

```
A. Place YAML or JSON `.config` files in the `.ebextensions/` directory at the root of the source bundle
B. Put the settings only in `requirements.txt`
C. Store the files in a `templates/` directory outside `.ebextensions/`
D. Configure a CloudFront Origin Group
```

### ✅ Answer: **A. Place YAML or JSON `.config` files in the `.ebextensions/` directory at the root of the source bundle**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- `.ebextensions/` nằm ở thư mục gốc của source code; các tệp `.config` dùng YAML hoặc JSON để thiết lập `option_settings` và có thể khai báo tài nguyên như RDS, ElastiCache hoặc DynamoDB.
- Tài nguyên do `.ebextensions` quản lý bị xóa khi Elastic Beanstalk environment bị xóa; không nên gắn dữ liệu production vào vòng đời environment.

**Tại sao loại các lựa chọn khác:**

| Option | Lý do loại bỏ |
|--------|---------------|
| B | `requirements.txt` liệt kê dependency Python, không phải nơi khai báo toàn bộ cấu hình Elastic Beanstalk. |
| C | Các tệp cấu hình phải nằm trong `.ebextensions/` ở gốc source bundle. |
| D | CloudFront Origin Group cấu hình failover giữa origins, không cấu hình Elastic Beanstalk environment. |

### 📝 Vocabulary Analysis:
- **`.ebextensions/`**: thư mục gốc chứa cấu hình mở rộng cho Elastic Beanstalk.
- **`option_settings`**: cấu hình các tùy chọn của platform/environment.
- **source bundle**: gói mã nguồn và tệp cấu hình được triển khai.

### 🔄 Context Variations:
- **If a database must survive environment deletion** → Create/manage it separately from the environment lifecycle and configure the application connection settings.

---
