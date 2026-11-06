# Giải thích bổ sung - Phần 7: CloudFront và Elastic Beanstalk

> *Phần này bổ sung kiến thức triển khai; nội dung mở rộng được tách khỏi bản dịch trang trình chiếu.*

---

### 1. Chọn origin và cách phân phối nội dung

#### Giải thích cốt lõi

CloudFront phục vụ nội dung từ Edge Locations, lưu cache để giảm số lần truy cập origin và độ trễ cho người dùng ở xa. **S3 Cross-Region Replication** tạo bản sao dữ liệu ở các Region được cấu hình; CloudFront phân phối nội dung từ cache toàn cầu. Vì vậy, CloudFront hợp với nội dung cần phân phối rộng, còn replication phù hợp khi ứng dụng cần bản dữ liệu ở một số Region.

Origin có thể là S3 hoặc HTTP backend như ALB/EC2. Origin Group dùng cặp origin chính/phụ để failover. VPC Origins cho phép CloudFront truy cập một số origin trong subnet riêng mà không công khai origin đó ra Internet.
Cập nhật 06/11/2026: AWS hiện công bố CloudFront có hơn 750 Points of Presence tại hơn 100 thành phố ở hơn 50 quốc gia, 15 Regional Edge Caches và hơn 1.140 điểm hiện diện tích hợp (embedded PoPs). Đây là số liệu hiện hành, tách khỏi số liệu trong trang trình chiếu. [Mạng Edge toàn cầu của CloudFront](https://aws.amazon.com/cloudfront/features/).


#### NÊN làm (DO)

- **Dùng CloudFront cho nội dung cần đọc từ nhiều vị trí địa lý** → Edge cache giảm số lượt trao đổi với origin. So với chỉ dùng S3 Cross-Region Replication, CloudFront phân phối từ edge và không cần cấu hình bản sao cho từng Region.
- **Dùng Origin Group khi có origin dự phòng tương thích** → CloudFront có thể chuyển sang origin phụ nếu origin chính gặp lỗi. So với một origin duy nhất, cách này giảm sự phụ thuộc vào một điểm phục vụ.
- **Dùng VPC Origins cho ALB/NLB/EC2 cần giữ trong subnet riêng** → Người dùng chỉ gọi CloudFront thay vì truy cập origin trực tiếp. So với đưa origin ra Internet, origin riêng tư giảm bề mặt truy cập.

#### KHÔNG NÊN làm (DON'T)

- **Dùng Cross-Region Replication chỉ để cache nội dung tĩnh trên toàn cầu** → Cần cấu hình sao chép theo Region nhưng vẫn chưa có mạng Edge cache. NÊN: dùng CloudFront để phân phối toàn cầu; chọn sao chép khi cần bản dữ liệu ở những Region cụ thể.
- **Đặt origin riêng tư trực tiếp trên Internet để CloudFront truy cập** → Tăng nguy cơ bị truy cập bên ngoài CloudFront. NÊN: dùng VPC Origins cho origin trong subnet riêng hoặc cấu hình quyền riêng cho S3 origin bằng OAC.

---

### 2. OAC, Cache Key và tỷ lệ cache hit

#### Giải thích cốt lõi

CloudFront dùng Cache Key để quyết định hai yêu cầu có thể dùng cùng đối tượng cache hay không. Mặc định Cache Key gồm hostname và đường dẫn tài nguyên; nếu nội dung thay đổi theo query string, cookie hoặc header, các thành phần đó cần được chọn trong Cache Policy. Thêm quá nhiều thành phần làm cache bị chia nhỏ và giảm Cache Hit Ratio.

Với S3 origin, Origin Access Control (OAC) cho phép CloudFront truy cập origin theo cấu hình kiểm soát thay vì để bucket công khai. OAC là phương thức được trang trình chiếu nêu để thay OAI.
OAC chỉ hỗ trợ S3 REST origin, không dùng được với S3 website endpoint. Nếu cần bucket riêng tư, dùng REST endpoint cùng OAC; website endpoint không đáp ứng mô hình OAC.

#### NÊN làm (DO)

- **Chỉ đưa header, cookie hoặc query string thực sự làm thay đổi nội dung vào Cache Key** → Cấu hình đúng giúp cache trả đúng phiên bản cho người dùng. So với đưa mọi thuộc tính yêu cầu vào key, cách này giữ Cache Hit Ratio cao hơn.
- **Dùng OAC để giới hạn truy cập S3 origin cho CloudFront** → Bucket vẫn có thể giữ riêng tư trong khi CloudFront distribution đọc được object. So với bucket công khai hoặc cấu hình OAI cũ, OAC là phương thức mới hơn theo trang trình chiếu.
- **Kiểm tra Cache Key bằng các yêu cầu đại diện cho từng biến thể nội dung** → Thiếu một thuộc tính có thể trả nhầm nội dung; đưa thừa thuộc tính gây cache miss. So với chỉ xem cấu hình, thử nhiều URL/header/cookie để xác nhận hành vi thực tế.

#### KHÔNG NÊN làm (DON'T)

- **Bỏ query string/header/cookie đang làm thay đổi nội dung khỏi Cache Key** → Người dùng có thể nhận bản cache của biến thể khác. NÊN: thêm đúng thuộc tính vào Cache Policy.
- **Đưa mọi cookie và header vào Cache Key theo mặc định** → Mỗi biến thể tạo mục cache riêng, làm giảm cache hit và tăng tải origin. NÊN: chỉ đưa vào các trường ảnh hưởng đến nội dung.

---

### 3. TTL, Cache Behaviors và invalidation

#### Giải thích cốt lõi

CloudFront giữ bản cache đến khi TTL hết hạn; thay đổi ở origin không tự làm mới bản đang cache. Invalidation yêu cầu CloudFront loại bỏ đường dẫn cụ thể hoặc toàn bộ cache để lần đọc sau lấy nội dung mới. Cache Behaviors áp dụng chính sách khác nhau theo path và có thể chọn origin/origin group; default behavior `/*` được xử lý cuối.

#### NÊN làm (DO)

- **Đặt TTL theo tốc độ thay đổi và yêu cầu mới của nội dung** → TTL dài tăng khả năng cache hit nhưng làm nội dung cũ tồn tại lâu hơn; TTL ngắn cập nhật nhanh hơn nhưng tăng request về origin. So với một TTL chung, tách behavior theo `/images/*`, `/api/*` giúp áp dụng chính sách khác nhau.
- **Dùng invalidation cho đường dẫn vừa thay đổi khi không thể chờ TTL** → Invalidation làm mới cache theo path cần thiết. So với invalidate `/*`, invalidate có mục tiêu tránh xóa các mục cache không liên quan.
- **Giữ default behavior ở cuối và kiểm thử thứ tự path patterns** → Behavior cụ thể cần được khớp trước `/*`. So với dựa vào một rule mặc định, behavior riêng giúp định tuyến đúng API, ảnh và nội dung tĩnh.

#### KHÔNG NÊN làm (DON'T)

- **Cho rằng cập nhật origin sẽ tự cập nhật cache CloudFront ngay lập tức** → CloudFront có thể tiếp tục trả nội dung cũ đến khi TTL hết hạn. NÊN: chờ TTL hoặc tạo invalidation cho path đã đổi.
- **Invalidate toàn bộ cache sau mỗi lần phát hành nhỏ** → Làm mất nhiều cache hit và tăng tải origin. NÊN: invalidate các đường dẫn bị ảnh hưởng hoặc dùng versioned file names khi quy trình ứng dụng cho phép.

---

### 4. Kiểm soát truy cập, vị trí và dữ liệu nhạy cảm

#### Giải thích cốt lõi

Geo Restriction giới hạn người xem theo quốc gia, còn Signed URL/Cookie giới hạn quyền truy cập nội dung riêng tư theo policy như thời hạn, dải IP và signer được tin cậy. Signed URL phù hợp một tệp; Signed Cookie cấp quyền cho nhiều tệp. Field-Level Encryption mã hóa các trường nhạy cảm trong POST request tại edge bằng public key; HTTPS vẫn cần để bảo vệ đường truyền.

Để tạo Signed URL, dùng private key tương ứng với public key nằm trong trusted key group mà distribution tin cậy; ký policy bằng RSA hoặc ECDSA kết hợp hàm băm SHA-1 hoặc SHA-256. Đây là chữ ký số để xác thực policy, không phải HMAC và không mã hóa nội dung. Cập nhật 06/11/2026: xem [hướng dẫn AWS tạo chữ ký cho policy tùy chỉnh](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-creating-signed-url-custom-policy.html#private-content-custom-policy-creating-signature).

#### NÊN làm (DO)

- **Dùng Signed URL cho một tệp trả phí và Signed Cookie cho nhóm tệp** → Cơ chế cấp quyền khớp phạm vi nội dung. So với public URL, link/cookie có policy giới hạn thời hạn và IP.
- **Giữ thời hạn Signed URL ngắn cho nội dung chia sẻ** → Link bị lộ sẽ mất hiệu lực sớm hơn. So với URL sống lâu, thời hạn ngắn giảm cửa sổ truy cập không mong muốn.
- **Dùng Field-Level Encryption khi backend cần bảo vệ một số trường POST xuyên suốt ứng dụng** → Trường được mã hóa tại edge bằng public key và backend có thể giải mã bằng private key. So với chỉ dựa vào HTTPS, dữ liệu đã chọn vẫn được mã hóa sau khi đi qua các tầng trung gian.
- **Dùng Geo Restriction khi yêu cầu bản quyền gắn với quốc gia** → Allowlist/Blocklist kiểm soát quốc gia được phục vụ. So với tự viết bộ lọc ở origin, restriction được thực hiện trên CloudFront.

#### KHÔNG NÊN làm (DON'T)

- **Dùng Geo Restriction như cơ chế xác thực người dùng** → Vị trí được suy ra từ cơ sở dữ liệu Geo-IP, không chứng minh danh tính. NÊN: dùng Signed URL/Cookie hoặc cơ chế xác thực phù hợp cho nội dung riêng tư.
- **Cho phép Signed URL tồn tại lâu hơn nhu cầu truy cập** → URL bị chia sẻ vẫn có hiệu lực trong thời hạn policy. NÊN: đặt expiry ngắn cho nội dung chia sẻ và giới hạn IP khi yêu cầu.
- **Coi Field-Level Encryption thay thế HTTPS** → HTTPS bảo vệ đường truyền; Field-Level Encryption chỉ mã hóa các trường đã chọn. NÊN: bật cả HTTPS và mã hóa trường nhạy cảm khi cần.

---

### 5. Chọn phương thức triển khai Elastic Beanstalk

#### Giải thích cốt lõi

Các phương thức triển khai đánh đổi giữa gián đoạn dịch vụ, năng lực phục vụ trong lúc triển khai, thời gian và chi phí. All at once nhanh nhất nhưng gây gián đoạn dịch vụ. Rolling cập nhật theo từng đợt nhưng tạm giảm năng lực phục vụ. Rolling with additional batches giữ nguyên năng lực phục vụ nhờ instance bổ sung. Immutable dựng ASG mới để kiểm tra phiên bản trước khi chuyển; Blue/Green tạo environment riêng; Traffic Splitting gửi một phần lưu lượng đến phiên bản mới để kiểm thử.

#### NÊN làm (DO)

- **Dùng Immutable cho production khi cần chuyển phiên bản không gián đoạn và rollback nhanh** → Phiên bản mới chạy trên ASG tạm; nếu lỗi, xóa ASG mới để quay lại. So với All at once, cách này an toàn hơn nhưng cần thêm capacity/chi phí tạm thời.
- **Dùng Rolling with additional batches khi cần giữ đủ capacity và chấp nhận triển khai lâu hơn** → Instance bổ sung duy trì năng lực phục vụ trong lúc thay batch. So với Rolling thường, tránh chạy dưới capacity nhưng tốn thêm chi phí.
- **Dùng Traffic Splitting để thử phiên bản mới với một tỷ lệ nhỏ người dùng** → Theo dõi tình trạng triển khai và có thể rollback tự động. So với chuyển toàn bộ lưu lượng ngay, canary giới hạn phạm vi ảnh hưởng.
- **Dùng Blue/Green bằng hai environment khi cần kiểm tra độc lập trước khi chuyển đổi** → Có thể kiểm thử environment mới rồi đổi URL/định tuyến. So với Rolling, hai phiên bản được tách trong hai environment riêng.

#### KHÔNG NÊN làm (DON'T)

- **Chọn All at once cho production cần tránh gián đoạn dịch vụ** → Instance có thể tạm thời không phục vụ lưu lượng. NÊN: dùng Immutable, Rolling with additional batches hoặc Blue/Green tùy chi phí và yêu cầu.
- **Nhầm Blue/Green là một phương thức triển khai trực tiếp của Elastic Beanstalk** → Trang trình chiếu nêu đây không phải tính năng trực tiếp; nếu tưởng tính năng này có sẵn, sẽ thiếu bước tạo environment và chuyển lưu lượng. NÊN: tạo environment mới, kiểm tra rồi đổi URL/Route 53.

---

### 6. Vòng đời environment, extensions và RDS

#### Giải thích cốt lõi

Elastic Beanstalk quản lý environment gồm phiên bản ứng dụng và tài nguyên nền, nhưng người vận hành vẫn cần hiểu vòng đời tài nguyên đó. `.ebextensions` cấu hình tài nguyên/setting khi tạo environment; tài nguyên do phần mở rộng quản lý có thể bị xóa cùng environment. Vì vậy, database production không nên phụ thuộc vòng đời environment như một RDS instance được tạo bên trong Beanstalk.

#### NÊN làm (DO)

- **Dùng lifecycle policy để dọn phiên bản ứng dụng cũ** → Mỗi application có giới hạn lưu 1.000 phiên bản; hết chỗ có thể không triển khai được. So với xóa thủ công tùy tiện, policy theo thời gian/dung lượng giữ lại phiên bản đang chạy và tự dọn bản cũ.
- **Dùng `.ebextensions/` cho cấu hình có thể lặp lại và xem xét cùng mã ứng dụng** → Tệp `.config` YAML/JSON giúp thiết lập cấu hình/tài nguyên cùng environment. So với chỉ thao tác trên Console, cấu hình bằng mã dễ tái tạo hơn.
- **Tách RDS production khỏi vòng đời Elastic Beanstalk** → Xóa environment có thể xóa tài nguyên do Beanstalk/CloudFormation quản lý. So với RDS gắn trong environment, database riêng tồn tại độc lập và có thể được dùng từ nhiều phiên bản.
- **Tạo environment mới khi cần đổi loại Load Balancer** → Trang trình chiếu nêu không thể đổi loại sau khi tạo environment. So với cố sửa environment cũ, tạo mới rồi đổi CNAME/Route 53 là quy trình di chuyển được mô tả.

#### KHÔNG NÊN làm (DON'T)

- **Để database production phụ thuộc vòng đời environment** → Kết thúc environment có thể ảnh hưởng database và tăng nguy cơ mất dữ liệu. NÊN: tạo RDS riêng, tạo snapshot và bật deletion protection trước khi di chuyển.
- **Giữ vô hạn mọi phiên bản ứng dụng hoặc xóa source bundle khi chưa có bản lưu khác** → Có thể chạm giới hạn 1.000 phiên bản hoặc mất gói mã nguồn. NÊN: đặt lifecycle policy và giữ source bundle cần cho rollback/khôi phục.
- **Cho rằng clone environment sao chép dữ liệu RDS** → Trang trình chiếu nêu clone giữ loại/cấu hình database nhưng không sao chép dữ liệu. NÊN: tạo snapshot/backup và kiểm tra dữ liệu riêng trước khi dùng bản clone.

### 7. OAC bucket policy và điều kiện Origin Group failover

#### Giải thích cốt lõi

OAC không tự biến bucket thành riêng tư: bucket policy phải cho phép CloudFront distribution cụ thể đọc object, đồng thời không cấp public read. AWS khuyến nghị OAC thay cho OAI; ví dụ policy ràng buộc `Principal` là `cloudfront.amazonaws.com` và `AWS:SourceArn` là ARN của distribution. Origin Group chỉ failover khi response của primary khớp tiêu chí failover được cấu hình; không nên hiểu là mọi lỗi ứng dụng đều tự động chuyển sang secondary. Theo tài liệu AWS được kiểm tra ngày 2026-11-06: [Restrict access to an Amazon S3 origin](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html), [Origin failover](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/high_availability_origin_failover.html).
Origin failover chỉ áp dụng cho viewer request dùng `GET`, `HEAD` hoặc `OPTIONS`; `POST`/`PUT` không failover. `OPTIONS` cần được bật trong Cached HTTP methods. Các failover status code hiện được chọn trong nhóm `400`, `403`, `404`, `416`, `429`, `500`, `502`, `503`, `504`; lỗi kết nối dùng tiêu chí `503`, timeout dùng `504`. [Điều kiện failover](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/high_availability_origin_failover.html).

#### NÊN làm (DO)

- **Cấp quyền bucket policy cho đúng distribution theo OAC** → `AWS:SourceArn` giới hạn distribution được đọc; so với public bucket, tránh cấp quyền đọc toàn Internet.
- **Cấu hình failover response codes phù hợp với dịch vụ và kiểm thử cả hai origin** → Failover chỉ diễn ra với điều kiện cấu hình; so với suy đoán mọi lỗi đều kích hoạt failover, kiểm thử mã lỗi giúp xác nhận hành vi thực tế.

#### KHÔNG NÊN làm (DON'T)

- **Chỉ tạo OAC mà bỏ qua bucket policy** → CloudFront không có quyền đọc object và trả lỗi truy cập. NÊN: cập nhật policy sau khi tạo distribution/OAC theo mẫu AWS.
- **Đưa toàn bộ lỗi ứng dụng vào failover mà không kiểm tra nội dung secondary** → Người dùng có thể nhận nội dung không nhất quán. NÊN: chỉ chọn mã lỗi và nguồn dự phòng tương thích với yêu cầu.
