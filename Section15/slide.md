# Section 15 - AWS CI/CD: CodeCommit, CodePipeline, CodeBuild, CodeDeploy và các dịch vụ liên quan (Trang 763-798)

> *Section 15 tập trung tự động hóa quy trình đưa mã từ repository qua build/test đến deploy, với khả năng kiểm soát và tăng độ an toàn cho phát hành.*

---

## Giới thiệu CI/CD (Trang 763-765)

- Phần trước đã giới thiệu cách tạo tài nguyên AWS thủ công, tương tác với AWS bằng CLI và triển khai mã lên Elastic Beanstalk.
- Thực hiện thủ công nhiều bước làm tăng khả năng xảy ra lỗi.
- Mục tiêu là lưu mã trong repository và tự động triển khai lên AWS theo quy trình phù hợp:
  - Kiểm thử trước khi triển khai.
  - Có thể triển khai qua các môi trường dev, test, staging và prod.
  - Có thể yêu cầu phê duyệt thủ công khi cần.
- Để trở thành AWS developer đúng nghĩa, cần học cách tự động hóa CI/CD.
- Section này trình bày:
  - **AWS CodeCommit:** lưu mã nguồn.
  - **AWS CodePipeline:** tự động hóa pipeline từ mã nguồn đến Elastic Beanstalk và các đích triển khai khác.
  - **AWS CodeBuild:** build và kiểm thử mã.
  - **AWS CodeDeploy:** triển khai mã lên EC2 (không phải Elastic Beanstalk), Lambda và các nền tảng được hỗ trợ.
  - **AWS CodeArtifact:** lưu trữ, publish và chia sẻ software package.
  - **AWS CodeGuru:** review mã tự động và đưa ra khuyến nghị hiệu năng bằng Machine Learning.

### Continuous Integration (CI) (Trang 766)

- Developer thường xuyên push mã lên code repository như GitHub, CodeCommit hoặc Bitbucket.
- Ngay khi có thay đổi, build/test server như CodeBuild hoặc Jenkins CI kiểm tra mã.
- Developer nhận phản hồi về các kiểm tra đã đạt hoặc thất bại.
- Lợi ích:
  - Phát hiện và sửa lỗi sớm.
  - Phân phối phần mềm nhanh hơn nhờ kiểm thử thường xuyên.
  - Triển khai thường xuyên.
  - Giảm thời gian developer bị chặn chờ.

### Continuous Delivery (CD) (Trang 767)

- Đảm bảo phần mềm luôn có thể được phát hành một cách tin cậy khi cần.
- Hỗ trợ triển khai thường xuyên và nhanh chóng.
- Chuyển từ phát hành ba tháng một lần sang phát hành năm lần mỗi ngày.
- Thường cần tự động hóa deploy bằng CodeDeploy, Jenkins CD, Spinnaker hoặc công cụ tương tự.

### Công nghệ trong CI/CD (Trang 768)

- CodeCommit hoặc repository bên thứ ba cung cấp mã nguồn.
- CodeBuild hoặc CI server bên thứ ba thực hiện build và test.
- CodePipeline điều phối các bước trong luồng.
- CodePipeline có thể điều phối quy trình đến Elastic Beanstalk; CodeDeploy triển khai riêng lên EC2 instance, on-premises instance, Lambda hoặc Amazon ECS.

> *Sơ đồ (Trang 768): Luồng từ code repository qua build/test đến deploy/provision; CodePipeline điều phối các thành phần AWS hoặc bên thứ ba.*

## AWS CodeCommit (Trang 769-772)

### Git và CodeCommit (Trang 769-770)

- Version control giúp theo dõi các thay đổi mã theo thời gian và có thể quay lại phiên bản trước.
- Git là một version control system phổ biến; repository có thể đồng bộ trên máy developer và được lưu trên repository tập trung trực tuyến.
- Lợi ích: cộng tác, sao lưu mã và cho phép xem lại/audit thay đổi.
- Tự vận hành Git repository có thể tốn kém; các lựa chọn phổ biến gồm GitHub, GitLab và Bitbucket.
- CodeCommit cung cấp Git repository riêng tư, được AWS quản lý và có tính sẵn sàng cao.
- Theo slide, repository có thể mở rộng mà không giới hạn kích thước; mã được lưu trong tài khoản AWS để hỗ trợ yêu cầu bảo mật/tuân thủ.
- CodeCommit hỗ trợ bảo mật và tích hợp với Jenkins, CodeBuild cùng các CI tool khác.

### Bảo mật CodeCommit (Trang 771)

- Tương tác repository bằng Git.
- **Xác thực:** SSH key; hoặc HTTPS qua AWS CLI credential helper/Git credentials cho IAM user.
- **Phân quyền:** IAM policy kiểm soát quyền của user/role trên repository.
- **Mã hóa:** repository được mã hóa khi lưu bằng AWS KMS và mã hóa khi truyền qua HTTPS hoặc SSH.
- **Truy cập cross-account:** không chia sẻ SSH key hoặc AWS credentials; dùng IAM Role và AWS STS `AssumeRole`.

### CodeCommit và GitHub (Trang 772)

| Đặc điểm trên slide | CodeCommit | GitHub |
|---|---|---|
| Code review qua pull request | Hỗ trợ | Hỗ trợ |
| Tích hợp CodeBuild | Hỗ trợ | Hỗ trợ |
| Xác thực SSH và HTTPS | Hỗ trợ | Hỗ trợ |
| Bảo mật | IAM user và role | GitHub user |
| Hosting | AWS quản lý và host | GitHub host; GitHub Enterprise có thể self-host trên server riêng |
| Giao diện | Tối giản | Đầy đủ tính năng |

## AWS CodePipeline (Trang 773-775)

### Pipeline và stages (Trang 773)

- CodePipeline là workflow trực quan để điều phối CI/CD.
- Có thể kết nối các loại action sau:
  - **Source:** CodeCommit, ECR, S3, Bitbucket, GitHub.
  - **Build:** CodeBuild, Jenkins, CloudBees, TeamCity.
  - **Test:** CodeBuild, AWS Device Farm và công cụ bên thứ ba.
  - **Deploy:** CodeDeploy, Elastic Beanstalk, CloudFormation, ECS, S3 và các đích khác.
  - **Invoke:** Lambda hoặc Step Functions.
- Pipeline gồm nhiều stage; mỗi stage có thể có các action chạy tuần tự hoặc song song, ví dụ Build, Test, Deploy và Load Testing.
- Có thể đặt manual approval ở bất kỳ stage nào.

### Artifact trong CodePipeline (Trang 774)

- Mỗi stage có thể tạo artifact.
- Artifact được lưu trong S3 bucket và chuyển sang stage tiếp theo.

> *Sơ đồ (Trang 774): Artifact được truyền giữa các stage của CodePipeline qua S3 bucket.*

### Khắc phục sự cố CodePipeline (Trang 775)

- Dùng CloudWatch Events (nay là EventBridge) để theo dõi thay đổi trạng thái execution của pipeline, action hoặc stage; ví dụ tạo event khi pipeline thất bại hoặc stage bị hủy.
- Khi một stage thất bại, pipeline dừng; có thể xem thông tin lỗi trong Console.
- Nếu pipeline không thực hiện được action, kiểm tra IAM Service Role đã có đủ quyền trong IAM policy chưa.
- Dùng CloudTrail để audit AWS API call.

## AWS CodeBuild (Trang 776-780)

### Tổng quan và cấu hình project (Trang 776-778)

- CodeBuild là dịch vụ CI được quản lý hoàn toàn.
- Mở rộng liên tục; không cần quản lý/cấp phát server và không có build queue theo slide.
- Biên dịch mã nguồn, chạy test và tạo software package.
- Là lựa chọn thay thế cho công cụ build như Jenkins.
- Tính phí theo phút sử dụng compute resource.
- Dùng Docker bên dưới để tạo môi trường build có thể tái lập; có thể dùng Docker image dựng sẵn hoặc image tùy chỉnh.
- Bảo mật gồm mã hóa build artifact bằng KMS, IAM permission cho CodeBuild, VPC cho network security và CloudTrail ghi log API call.
- Source có thể là CodeCommit, S3, Bitbucket hoặc GitHub.
- Chỉ dẫn build được khai báo trong `buildspec.yml` hoặc nhập trực tiếp trên Console.
- Log có thể lưu vào S3 và CloudWatch Logs; CloudWatch Metrics theo dõi thống kê build.
- Dùng EventBridge phát hiện build thất bại và gửi thông báo; dùng CloudWatch Alarm khi cần ngưỡng cảnh báo.
- Build project có thể được định nghĩa trong CodePipeline hoặc trong CodeBuild.
- Môi trường được nêu trên slide: Java, Ruby, Python, Go, Node.js, Android, .NET Core, PHP và Docker.

> *Sơ đồ (Trang 779): CodeBuild lấy source code cùng `buildspec.yml`, chạy lệnh trong container; có thể dùng cache ở S3 và lưu artifact/log vào S3 hoặc CloudWatch Logs.*

### Tệp `buildspec.yml` (Trang 780)

- `buildspec.yml` phải nằm ở thư mục gốc của source code.
- `env` định nghĩa biến môi trường:
  - `variables`: giá trị plaintext.
  - `parameter-store`: giá trị lấy từ SSM Parameter Store.
  - `secrets-manager`: giá trị lấy từ AWS Secrets Manager.
- `phases` khai báo lệnh theo giai đoạn:
  - `install`: cài dependency cần thiết.
  - `pre_build`: lệnh chuẩn bị cuối trước build.
  - `build`: lệnh build chính.
  - `post_build`: bước hoàn tất, ví dụ đóng gói ZIP.
- `artifacts` khai báo nội dung upload lên S3; artifact được mã hóa bằng KMS.
- `cache` khai báo tệp cần lưu trên S3 để tăng tốc build sau, thường là dependency.

## AWS CodeDeploy (Trang 781-790)

### Nền tảng và chiến lược EC2/on-premises (Trang 781-784)

- CodeDeploy tự động hóa triển khai phiên bản ứng dụng mới lên EC2 instance, on-premises server, Lambda function hoặc ECS service.
- Hỗ trợ rollback tự động khi deploy thất bại hoặc khi CloudWatch Alarm được kích hoạt.
- Có thể điều khiển tốc độ triển khai.
- Tệp `appspec.yml` mô tả cách thực hiện deployment.
- Với EC2/on-premises, có thể triển khai **in-place** hoặc **blue/green**.
- EC2/on-premises deployment yêu cầu CodeDeploy Agent chạy trên target instance.
- Các cấu hình tốc độ triển khai:
  - `AllAtOnce`: triển khai lên tất cả instance cùng lúc; nhanh nhất nhưng có nguy cơ downtime cao nhất.
  - `HalfAtATime`: triển khai lên một nửa số instance mỗi lần; dung lượng phục vụ giảm khoảng 50% trong giai đoạn đó.
  - `OneAtATime`: lần lượt triển khai từng instance; chậm nhất nhưng ảnh hưởng năng lực phục vụ thấp nhất.
  - `Custom`: tự xác định tỷ lệ instance được triển khai trong mỗi lượt.

> *Sơ đồ (Trang 783): Minh họa quá trình in-place deployment trên các EC2 instance hiện có.*
>
> *Sơ đồ (Trang 784): Blue/green giữ hai môi trường để chuyển traffic từ môi trường cũ sang môi trường mới.*

### CodeDeploy Agent (Trang 785)

- CodeDeploy Agent phải chạy trên EC2 instance đích trước khi triển khai.
- Có thể cài đặt và cập nhật Agent tự động bằng Systems Manager.
- EC2 instance phải có đủ quyền truy cập S3 để lấy deployment bundle.

### Triển khai Lambda và ECS (Trang 786-787)

- Với Lambda, CodeDeploy có thể tự động chuyển traffic giữa các alias; tính năng này tích hợp với SAM.
- Chiến lược Lambda:
  - **Linear:** tăng traffic theo từng bước, ví dụ `LambdaLinear10PercentEvery3Minutes` hoặc `LambdaLinear10PercentEvery10Minutes`.
  - **Canary:** chuyển một tỷ lệ nhỏ trước rồi chuyển 100%, ví dụ `LambdaCanary10Percent5Minutes` hoặc `LambdaCanary10Percent30Minutes`.
  - **AllAtOnce:** chuyển traffic ngay lập tức.
- Với ECS, CodeDeploy triển khai task definition mới và chỉ hỗ trợ blue/green deployment.
- Chiến lược ECS gồm `ECSLinear10PercentEvery3Minutes`, `ECSLinear10PercentEvery10Minutes`, `ECSCanary10Percent5Minutes`, `ECSCanary10Percent30Minutes` và `AllAtOnce`.

### Triển khai EC2 và Auto Scaling Group (Trang 788-789)

- Deployment lên EC2 được xác định bởi `appspec.yml` và deployment strategy; có thể dùng hook để xác minh sau từng phase.
- Với Auto Scaling Group:
  - **In-place:** cập nhật các instance hiện tại; instance mới do ASG tạo cũng được tự động triển khai ứng dụng.
  - **Blue/green:** tạo ASG mới với cấu hình được sao chép; chọn khoảng thời gian giữ ASG cũ; cần dùng Elastic Load Balancer.

### Redeploy và rollback (Trang 790)

- Rollback là triển khai lại revision ứng dụng đã triển khai trước đó.
- Có thể rollback tự động khi deployment thất bại hoặc khi CloudWatch Alarm vượt ngưỡng; cũng có thể rollback thủ công.
- Có thể tắt rollback cho deployment.
- Khi rollback, CodeDeploy triển khai lại revision tốt gần nhất như một deployment mới, không khôi phục một version deployment cũ.

## AWS CodeArtifact (Trang 791-794)

### Quản lý package và dependency (Trang 791-792)

- Software package phụ thuộc vào các package khác để build; các package được tạo ra cũng cần được lưu và truy xuất. Việc này gọi là artifact management.
- CodeArtifact là dịch vụ quản lý artifact bảo mật, có khả năng mở rộng và tiết kiệm chi phí.
- Hỗ trợ công cụ quản lý dependency như Maven, Gradle, npm, yarn, twine, pip và NuGet.
- Developer và CodeBuild có thể truy xuất dependency từ CodeArtifact.

> *Sơ đồ (Trang 792): Domain/repository CodeArtifact có thể proxy public repository như npm, NuGet, PyPI và Maven. IT Lead publish/approve package; developer dùng npm, pip, NuGet hoặc Maven để truy xuất; CodeBuild cũng lấy dependency từ CodeArtifact.*

### Tích hợp CodeArtifact với EventBridge (Trang 793)

- Event được tạo khi package version được tạo, sửa đổi hoặc xóa.
- EventBridge có thể kích hoạt Lambda, Step Functions, SNS, SQS hoặc CodePipeline.
- Ví dụ: khởi chạy pipeline CodeCommit → CodeBuild → CodeDeploy để build và triển khai lại ứng dụng với security fix mới nhất.

### Resource policy của CodeArtifact (Trang 794)

- Có thể dùng resource policy để cho phép account khác truy cập CodeArtifact.
- Theo slide, một principal được đọc toàn bộ package trong repository hoặc không package nào.

## Amazon CodeGuru (Trang 795-798)

### Tổng quan (Trang 795)

- Dịch vụ dùng Machine Learning để review code tự động và đưa ra khuyến nghị hiệu năng ứng dụng.
- Có hai chức năng:
  - **CodeGuru Reviewer:** review code tĩnh trong giai đoạn phát triển.
  - **CodeGuru Profiler:** quan sát runtime và đề xuất tối ưu trong production.

### CodeGuru Reviewer (Trang 796)

- Tìm vấn đề nghiêm trọng, lỗ hổng bảo mật và bug khó phát hiện.
- Ví dụ: vi phạm best practice, rò rỉ resource, vấn đề security và kiểm tra input.
- Dùng Machine Learning và automated reasoning, dựa trên kinh nghiệm từ hàng triệu code review trên hàng nghìn repository mã nguồn mở và của Amazon.
- Hỗ trợ Java và Python.
- Tích hợp GitHub, Bitbucket và CodeCommit.

### CodeGuru Profiler (Trang 797)

- Giúp hiểu hành vi runtime của ứng dụng, ví dụ tìm routine logging tiêu thụ CPU quá mức.
- Có thể phát hiện và loại bỏ code kém hiệu quả, cải thiện hiệu năng/giảm CPU, giảm chi phí compute, dùng heap summary để nhận diện object tiêu thụ nhiều bộ nhớ và phát hiện anomaly.
- Hỗ trợ ứng dụng chạy trên AWS hoặc on-premises; có overhead thấp.

### Cấu hình CodeGuru Profiler Agent (Trang 798)

- `MaxStackDepth`: độ sâu tối đa của call stack được đưa vào profile. Ví dụ call chain A → B → C → D có độ sâu 4; nếu đặt 2 thì profiler đánh giá A và B.
- `MemoryUsageLimitPercent`: tỷ lệ bộ nhớ tối đa profiler sử dụng.
- `MinimumTimeForReportingInMilliseconds`: thời gian tối thiểu giữa các lần gửi report.
- `ReportingIntervalInMilliseconds`: khoảng thời gian gửi profile report.
- `SamplingIntervalInMilliseconds`: khoảng cách giữa các lần lấy mẫu; giảm giá trị này để tăng sampling rate.

### Bài tập trên slide (sau Trang 798)

- Hướng dẫn từng bước tạo CI/CD pipeline bằng các dịch vụ AWS.
