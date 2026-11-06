# Phần 7 - AWS: CloudFront và Elastic Beanstalk (Trang 337-384)

---

## Phần 1: Amazon CloudFront (Trang 337-358)

### CloudFront và điểm gốc (Trang 338-342)

- **CloudFront** là mạng phân phối nội dung (CDN), cải thiện tốc độ đọc bằng cách lưu nội dung tại các Edge Locations và giảm độ trễ cho người dùng.
- Trang trình chiếu nêu mạng lưới có hơn **600 điểm hiện diện (Points of Presence)**; CloudFront tích hợp với **Shield** và **WAF** để hỗ trợ chống DDoS.
- **Origin** là nơi CloudFront lấy nội dung:
  - **S3 bucket**: phân phối tệp và lưu cache tại Edge Location; **Origin Access Control (OAC)** tăng cường bảo mật và thay thế **Origin Access Identity (OAI)**.
  - CloudFront cũng có thể làm điểm nhận để tải tệp lên S3.
  - **Custom HTTP origin**: Application Load Balancer, EC2 instance, S3 static website đã bật chế độ website hoặc HTTP backend khác.

> *Sơ đồ (Trang 341): CloudFront nhận yêu cầu tại Edge Location rồi lấy nội dung từ origin.*
> *Sơ đồ (Trang 342): CloudFront phân phối object từ S3 origin tới các Edge Locations; Origin Access Control (OAC) phối hợp với S3 bucket policy để giới hạn quyền truy cập bucket.*


- **Origin Group** gồm một origin chính và một origin phụ; nếu origin chính gặp lỗi phù hợp điều kiện failover, CloudFront chuyển sang origin phụ để tăng tính sẵn sàng.

### CloudFront và sao chép liên vùng S3 (Trang 343-344)

| Đặc điểm | CloudFront | S3 Cross-Region Replication |
|---------|------------|------------------------------|
| Phạm vi | Mạng Edge Location toàn cầu | Các Region được cấu hình sao chép |
| Cách cập nhật | Lưu cache theo TTL | Sao chép tệp gần thời gian thực; bản sao chỉ đọc |
| Phù hợp | Nội dung tĩnh cần phân phối rộng | Nội dung động cần độ trễ thấp tại một số Region |

### Cache, Cache Key và hành vi cache (Trang 345-348)

- Cache nằm tại từng CloudFront Edge Location; mỗi đối tượng trong cache được nhận diện bằng **Cache Key**.
- Mặc định, Cache Key gồm hostname và đường dẫn tài nguyên trong URL.
- Nếu nội dung thay đổi theo người dùng, thiết bị, ngôn ngữ hoặc vị trí, có thể thêm HTTP header, cookie và query string bằng **Cache Policy**.
- Nên tăng **Cache Hit Ratio** để giảm số yêu cầu gửi đến origin.
- Khi origin được cập nhật, CloudFront không tự nhận biết nội dung đã thay đổi; cache chỉ hết hạn sau TTL. Có thể gọi API `CreateInvalidation` để làm mới toàn bộ cache (`*`) hoặc một đường dẫn cụ thể.
- **Cache Behavior** áp dụng các thiết lập khác nhau cho mẫu đường dẫn, ví dụ `/images/*` hoặc `/api/*`, và có thể định tuyến đến origin hoặc Origin Group khác nhau.
- **Default Cache Behavior** luôn được xử lý cuối cùng và khớp với `/*`.

### VPC Origins (Trang 349-350)

- **VPC Origins** cho phép CloudFront phân phối nội dung từ ứng dụng trong private subnet mà không cần công khai ứng dụng trên Internet.
- Các loại origin riêng tư được liệt kê trên trang trình chiếu: Application Load Balancer, Network Load Balancer và EC2 instance.

> *Sơ đồ (Trang 350): Với EC2 làm origin, instance cần public IP; với ALB làm origin, ALB phải public nhưng EC2 phía sau có thể private. Đây là ví dụ origin công khai, không phải sơ đồ CloudFront VPC Origins.*

### Giới hạn theo vị trí địa lý và nội dung có kiểm soát truy cập (Trang 351-354)

- **Geo Restriction** giới hạn người xem theo quốc gia:
  - **Allowlist**: chỉ cho phép các quốc gia trong danh sách.
  - **Blocklist**: chặn các quốc gia trong danh sách.
- Quốc gia được xác định bằng cơ sở dữ liệu Geo-IP của bên thứ ba; trường hợp sử dụng gồm kiểm soát bản quyền nội dung.
- **Signed URL** và **Signed Cookie** dùng để phân phối nội dung trả phí hoặc riêng tư; policy có thể chứa thời hạn URL, dải IP được phép và bên ký được tin cậy.
  - Signed URL cấp quyền cho một tệp.
  - Signed Cookie cấp quyền cho nhiều tệp.
- Trang trình chiếu khuyến nghị thời hạn ngắn cho nội dung chia sẻ như phim/nhạc; nội dung riêng tư có thể cần thời hạn dài hơn.

> *Sơ đồ (Trang 353): Signed URL/Cookie cho phép người dùng được cấp quyền truy cập nội dung CloudFront.*

- **Field-Level Encryption** bổ sung một lớp bảo vệ ngoài HTTPS: mã hóa bất đối xứng các trường nhạy cảm trong yêu cầu POST tại Edge Location gần người dùng.
- Có thể chỉ định tối đa **10 trường** cần mã hóa và public key dùng để mã hóa.

### Nhật ký thời gian thực và chi phí (Trang 355-358)

- CloudFront có thể gửi yêu cầu thời gian thực đến **Kinesis Data Streams** để giám sát, phân tích và xử lý hiệu năng phân phối nội dung.
- Có thể chọn tỷ lệ lấy mẫu, các trường cần gửi và Cache Behavior/đường dẫn cần theo dõi.
- Chi phí truyền dữ liệu ra khác nhau theo Edge Location.
- Có thể giảm chi phí bằng **Price Classes**:
  1. **Price Class All**: mọi khu vực, hiệu năng tốt nhất.
  2. **Price Class 200**: phần lớn khu vực, loại các khu vực đắt nhất.
  3. **Price Class 100**: chỉ các khu vực có chi phí thấp nhất.

> *Sơ đồ (Trang 358): Phạm vi Edge Location của Price Class 100, 200 và All.*

## Phần 2: AWS Elastic Beanstalk (Trang 359-384)

### Tổng quan và thành phần (Trang 359-364)

- Elastic Beanstalk giúp nhà phát triển triển khai ứng dụng trên AWS mà không phải tự quản lý từng bước cấp phát hạ tầng.
- Các khó khăn phổ biến gồm quản lý hạ tầng, triển khai mã, cấu hình cơ sở dữ liệu/cân bằng tải và xử lý tự động mở rộng.
- Nhiều ứng dụng web có kiến trúc ALB + ASG; nhà phát triển muốn tập trung vào mã và triển khai nhất quán giữa các ứng dụng, môi trường.
- Dịch vụ dùng các thành phần như EC2, Auto Scaling Group, Load Balancer và RDS; tự xử lý cấp phát capacity, cân bằng tải, tự động mở rộng, theo dõi tình trạng ứng dụng và cấu hình instance.
- Nhà phát triển chịu trách nhiệm về mã ứng dụng nhưng vẫn có quyền kiểm soát cấu hình hạ tầng.
- Elastic Beanstalk không tính phí dịch vụ riêng; người dùng trả phí cho các tài nguyên nền được tạo.
- Các thành phần:
  - **Application**: tập hợp environment, version và configuration.
  - **Application Version**: một phiên bản mã ứng dụng.
  - **Environment**: tập hợp tài nguyên AWS chạy một application version; tại một thời điểm chỉ chạy một version.
  - **Environment Tier**: Web Server và Worker; có thể tạo nhiều environment như dev, test, prod.
- Các nền tảng được liệt kê trên trang trình chiếu: Go, Java SE, Java với Tomcat, .NET Core trên Linux, .NET trên Windows Server, Node.js, PHP, Python, Ruby, Packer Builder, Single Container Docker, Multi-container Docker và Preconfigured Docker.

> *Sơ đồ (Trang 360): Kiến trúc ứng dụng web ba tầng điển hình.*
>
> *Sơ đồ (Trang 365): So sánh Web Server Environment Tier và Worker Environment Tier.*

### Các phương thức triển khai (Trang 366-373)

| Phương thức | Cách triển khai và đánh đổi được trình bày |
|------------|----------------------------------------|
| **All at once** | Cập nhật mọi instance cùng lúc; nhanh nhất nhưng gây gián đoạn dịch vụ; phù hợp môi trường dev; không phát sinh chi phí bổ sung. |
| **Rolling** | Cập nhật từng batch; ứng dụng tạm chạy dưới mức capacity và hai phiên bản cùng hoạt động; lâu hơn nhưng không phát sinh chi phí bổ sung. |
| **Rolling with additional batches** | Thêm batch instance để giữ nguyên capacity; hai phiên bản chạy đồng thời; tốn thêm ít chi phí và lâu hơn; batch bổ sung bị xóa khi hoàn tất; phù hợp production. |
| **Immutable** | Tạo ASG tạm và instance mới; chuyển toàn bộ khi đạt trạng thái khỏe; không gây gián đoạn dịch vụ, rollback nhanh bằng cách xóa ASG mới nhưng cần gấp đôi capacity trong lúc triển khai. |
| **Blue/Green** | Không phải tính năng triển khai trực tiếp của Elastic Beanstalk; tạo environment mới, kiểm tra rồi đổi URL hoặc chuyển lưu lượng bằng Route 53. |
| **Traffic Splitting** | Kiểm thử canary: triển khai version mới lên ASG tạm có cùng capacity, gửi một phần lưu lượng trong khoảng thời gian cấu hình và theo dõi tình trạng; tự rollback nếu triển khai thất bại. Không gây gián đoạn dịch vụ; chuyển instance mới sang ASG gốc rồi kết thúc version cũ. |

> *Sơ đồ (Trang 373): Traffic Splitting — triển khai canary bằng ASG tạm, theo dõi tình trạng và tự động rollback nếu triển khai thất bại.*

### Tóm tắt triển khai Elastic Beanstalk (Trang 374)

- Trang trình chiếu dẫn tài liệu AWS về triển khai phiên bản ứng dụng hiện có: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.deploy-existing-version.html

### EB CLI (Trang 375)

- **EB CLI** hỗ trợ làm việc với Elastic Beanstalk trong quy trình triển khai; các lệnh cơ bản gồm `eb create`, `eb status`, `eb health`, `eb events`, `eb logs`, `eb open`, `eb deploy`, `eb config` và `eb terminate`.

### Quy trình triển khai Elastic Beanstalk (Trang 376)

- Khai báo các gói phụ thuộc, ví dụ `requirements.txt` cho Python hoặc `package.json` cho Node.js.
- Đóng gói mã thành ZIP. Có thể tải ZIP lên Console để tạo phiên bản ứng dụng rồi triển khai, hoặc dùng CLI để tạo phiên bản và triển khai.
- Elastic Beanstalk phân phối ZIP đến các EC2 instance, cài các gói phụ thuộc và khởi động ứng dụng.

### Quản lý version và phần mở rộng (Trang 377-378)

- Elastic Beanstalk lưu tối đa **1.000 application version**. Nếu không dọn version cũ, có thể không triển khai thêm được.
- **Lifecycle Policy** xóa version cũ theo thời gian hoặc dung lượng; version đang được sử dụng không bị xóa. Có thể giữ lại source bundle trên S3 để tránh mất dữ liệu.
- **`.ebextensions/`** nằm ở thư mục gốc mã nguồn; chứa tệp `.config` dạng YAML/JSON để cấu hình `option_settings` hoặc thêm tài nguyên như RDS, ElastiCache và DynamoDB.
- Tài nguyên do `.ebextensions` quản lý sẽ bị xóa khi environment bị xóa.
- Elastic Beanstalk dùng CloudFormation bên dưới để cấp phát dịch vụ AWS; `.ebextensions` có thể khai báo tài nguyên CloudFormation.

### Clone environment, Load Balancer và RDS (Trang 379-383)

- Có thể clone environment để giữ cấu hình cho một môi trường kiểm thử. Cấu hình Load Balancer, loại RDS và biến môi trường được sao chép; dữ liệu trong RDS không được sao chép.
- Sau khi tạo environment, không thể đổi loại Elastic Load Balancer, chỉ đổi cấu hình. Để chuyển loại, tạo environment mới (không clone), triển khai ứng dụng rồi đổi CNAME hoặc cập nhật Route 53.
- Có thể tạo RDS cùng Beanstalk cho dev/test; không phù hợp production vì vòng đời database gắn với environment.
- Với production, tạo RDS riêng và cấu hình chuỗi kết nối cho ứng dụng.
- Quy trình tách RDS khỏi environment: tạo snapshot, bật deletion protection cho database, tạo environment mới không kèm RDS và trỏ đến database hiện có, đổi CNAME/Route 53 rồi xác nhận hoạt động; sau đó kết thúc environment cũ và xử lý CloudFormation stack ở trạng thái `DELETE_FAILED`.

## Bài tập trên trang trình chiếu (Trang 384)

- Triển khai Blue/Green với Elastic Beanstalk.
