# Section 14 - AWS Serverless: SAM, CDK, Cognito và các dịch vụ khác (Trang 703-760)

> *Section 14 gồm AWS Serverless Application Model (SAM), Cloud Development Kit (CDK), Cognito User Pools, Cognito Identity Pools, Cognito Sync và các dịch vụ serverless như Step Functions, AppSync, Amplify.*

---

## AWS Serverless Application Model (SAM) (Trang 703-715)
> *Đưa quá trình phát triển serverless lên cấp độ tiếp theo.*

### AWS SAM là gì? (Trang 704)

- SAM là viết tắt của **Serverless Application Model**.
- Framework dùng để phát triển và triển khai ứng dụng serverless.
- Toàn bộ cấu hình được viết bằng YAML.
- Tạo cấu hình CloudFormation phức tạp từ một tệp SAM YAML đơn giản.
- Hỗ trợ các khả năng của CloudFormation như Outputs, Mappings, Parameters và Resources.
- SAM có thể dùng CodeDeploy để triển khai Lambda function.
- SAM hỗ trợ chạy Lambda, API Gateway và DynamoDB trên máy cục bộ.

### Quy trình SAM (Trang 705)

- Trường `Transform` cho biết template sử dụng SAM:

  ```yaml
  Transform: 'AWS::Serverless-2016-10-31'
  ```

- Khai báo tài nguyên bằng các loại như:
  - `AWS::Serverless::Function`
  - `AWS::Serverless::Api`
  - `AWS::Serverless::SimpleTable`
- Đóng gói và triển khai bằng `sam deploy`; có thể chạy `sam package` trước đó.
- Dùng SAM Accelerate (`sam sync --watch`) để đồng bộ nhanh thay đổi cục bộ lên Lambda.

> *Sơ đồ (Trang 706): Quy trình triển khai ứng dụng bằng SAM.*

### SAM Accelerate (`sam sync`) (Trang 707-708)

- SAM Accelerate là tập hợp tính năng giúp giảm độ trễ khi triển khai tài nguyên.
- `sam sync` đồng bộ dự án được khai báo trong SAM template lên AWS.
- Đồng bộ thay đổi mã mà không cập nhật hạ tầng bằng CloudFormation; thao tác này dùng API của dịch vụ và bỏ qua CloudFormation.
- Các cách dùng:
  - `sam sync`: đồng bộ mã và hạ tầng.
  - `sam sync --code`: đồng bộ thay đổi mã mà không cập nhật hạ tầng; bỏ qua CloudFormation và hoàn tất nhanh.
  - `sam sync --code --resource AWS::Serverless::Function`: chỉ đồng bộ các Lambda function và phần phụ thuộc.
  - `sam sync --code --resource-id HelloWorldLambdaFunction`: chỉ đồng bộ tài nguyên có ID cụ thể.
  - `sam sync --watch`: theo dõi thay đổi tệp và tự động đồng bộ; thay đổi cấu hình dùng `sam sync`, còn thay đổi chỉ có mã dùng `sam sync --code`.

### Gỡ lỗi bằng SAM CLI (Trang 709)

- Dùng SAM CLI để build, kiểm thử và gỡ lỗi cục bộ cho ứng dụng serverless được khai báo bằng SAM template.
- Cung cấp môi trường thực thi cục bộ mô phỏng Lambda.
- Kết hợp SAM CLI với AWS Toolkits để gỡ lỗi từng bước.
- IDE được hỗ trợ gồm AWS Cloud9, Visual Studio Code, JetBrains, PyCharm và IntelliJ.
- AWS Toolkits là plugin IDE hỗ trợ build, kiểm thử, gỡ lỗi, triển khai và gọi Lambda được xây dựng bằng SAM.

### SAM policy templates (Trang 710)

- Danh sách template để gán quyền cho Lambda function.
- Ví dụ:
  - `S3ReadPolicy`: chỉ đọc object trong S3.
  - `SQSPollerPolicy`: cho phép poll một SQS queue.
  - `DynamoDBCrudPolicy`: cho phép CRUD (tạo, đọc, cập nhật, xóa) trên DynamoDB.

### SAM và CodeDeploy (Trang 711-712)

- SAM tích hợp CodeDeploy để cập nhật Lambda function.
- Hỗ trợ chuyển traffic theo từng giai đoạn.
- Có thể chạy hook trước và sau khi chuyển traffic để kiểm tra triển khai.
- Có thể tự động rollback dựa trên CloudWatch Alarm.
- `AutoPublishAlias`:
  - Phát hiện khi có mã mới được triển khai.
  - Tạo và publish version cập nhật của function.
  - Chuyển alias sang version mới.
- `DeploymentPreference` quy định chiến lược `Canary`, `Linear` hoặc `AllAtOnce`.
- Có thể cấu hình Alarm để kích hoạt rollback.
- Hook là Lambda function chạy trước hoặc sau quá trình chuyển traffic.

### Khả năng chạy cục bộ của SAM (Trang 713-714)

- `sam local start-lambda`:
  - Khởi động endpoint cục bộ mô phỏng Lambda.
  - Có thể chạy kiểm thử tự động qua endpoint này.
- `sam local invoke`:
  - Gọi Lambda một lần bằng payload rồi kết thúc khi invocation hoàn tất.
  - Hữu ích để tạo và kiểm thử test case.
  - Nếu function gọi API AWS, cần chọn đúng tùy chọn `--profile`.
- `sam local start-api`:
  - Khởi động HTTP server cục bộ cho các function.
  - Tự nạp lại khi mã function thay đổi.
- `sam local generate-event` tạo payload mẫu cho Lambda từ các nguồn như S3, API Gateway, SNS, Kinesis và DynamoDB.

> *Sơ đồ (Trang 715): Cấu hình SAM cho nhiều môi trường.*

## AWS Cloud Development Kit (CDK) (Trang 716-723)

### AWS CDK (Trang 717)

- CDK cho phép định nghĩa hạ tầng cloud bằng ngôn ngữ lập trình quen thuộc:
  - JavaScript/TypeScript, Python, Java và .NET.
- Các thành phần cấp cao của CDK được gọi là **construct**.
- Mã CDK được tổng hợp thành CloudFormation template dạng JSON/YAML.
- Có thể triển khai hạ tầng và mã runtime của ứng dụng cùng nhau; phù hợp với Lambda và Docker container trên ECS/EKS.

> *Sơ đồ (Trang 718): Các thành phần và luồng tổng hợp template của CDK.*

### So sánh CDK và SAM (Trang 719)

| SAM | CDK |
|---|---|
| Tập trung vào ứng dụng serverless. | Hỗ trợ nhiều dịch vụ AWS. |
| Khai báo template bằng JSON hoặc YAML. | Khai báo hạ tầng bằng JavaScript/TypeScript, Python, Java hoặc .NET. |
| Phù hợp để bắt đầu nhanh với Lambda. | Dùng ngôn ngữ lập trình để định nghĩa hạ tầng. |
| Dựa trên CloudFormation. | Dựa trên CloudFormation. |

### Dùng SAM CLI với CDK (Trang 720)

- Có thể dùng SAM CLI để kiểm thử cục bộ ứng dụng CDK.
- Trước tiên phải chạy `cdk synth`.

### Các lệnh CDK cần biết (Trang 721)

| Lệnh | Tác dụng theo slide |
|---|---|
| `npm install -g aws-cdk-lib` | Cài CDK CLI và thư viện. |
| `cdk init app` | Tạo dự án CDK mới từ template được chỉ định. |
| `cdk synth` | Tổng hợp và in CloudFormation template. |
| `cdk bootstrap` | Triển khai stack staging của CDK Toolkit. |
| `cdk deploy` | Triển khai các stack. |
| `cdk diff` | Xem khác biệt giữa CDK cục bộ và stack đã triển khai. |
| `cdk destroy` | Xóa các stack. |

### CDK bootstrap (Trang 722)

- Bootstrap là quá trình cấp phát tài nguyên cần thiết trước khi triển khai ứng dụng CDK vào một AWS environment.
- AWS environment gồm account và Region.
- CloudFormation stack `CDKToolkit` được tạo, chứa:
  - S3 bucket để lưu trữ tệp.
  - IAM Role để cấp quyền triển khai.
- Chạy lệnh sau cho mỗi environment mới:

  ```bash
  cdk bootstrap aws://<aws_account>/<aws_region>
  ```

- Nếu chưa bootstrap, có thể gặp lỗi: `Policy contains a statement with one or more invalid principal`.

### Kiểm thử CDK (Trang 723)

- Dùng CDK Assertions Module cùng framework như Jest (JavaScript) hoặc Pytest (Python).
- Có thể xác minh resource, rule, condition và parameter cụ thể.
- Hai loại kiểm thử:
  - **Fine-grained Assertions** (thường dùng): kiểm tra chi tiết của CloudFormation template, ví dụ resource có property với giá trị mong muốn.
  - **Snapshot Tests**: so sánh CloudFormation template được tổng hợp với template baseline đã lưu.
- Nạp template bằng `Template.fromStack(MyStack)` với stack tạo trong CDK hoặc `Template.fromString(mystring)` với template tạo bên ngoài CDK.

## Amazon Cognito (Trang 724-737)

### Tổng quan về Amazon Cognito (Trang 725)

- Cung cấp danh tính cho người dùng để tương tác với ứng dụng web hoặc mobile.
- **Cognito User Pools (CUP)**:
  - Cung cấp chức năng đăng nhập cho người dùng ứng dụng.
  - Tích hợp với API Gateway và Application Load Balancer.
- **Cognito Identity Pools (Federated Identity)**:
  - Cấp AWS credentials cho người dùng để truy cập trực tiếp tài nguyên AWS.
  - Có thể dùng Cognito User Pool làm identity provider.
- So với IAM, Cognito phù hợp với tình huống có hàng trăm người dùng, người dùng mobile hoặc xác thực qua SAML.

### Cognito User Pools: tính năng cho người dùng (Trang 726)

- Tạo kho người dùng serverless cho ứng dụng web và mobile.
- Đăng nhập bằng username hoặc email cùng password.
- Hỗ trợ đặt lại password, xác minh email/số điện thoại và MFA.
- Hỗ trợ đăng nhập liên kết qua Facebook, Google, SAML và các nguồn danh tính khác.
- Có thể chặn người dùng nếu thông tin xác thực của họ bị phát hiện đã bị lộ ở nơi khác.
- Sau khi đăng nhập, trả về JSON Web Token (JWT).

> *Sơ đồ (Trang 727): Luồng đăng nhập và token của Cognito User Pool.*

### Tích hợp Cognito User Pool (Trang 728)

- Cognito User Pool tích hợp với API Gateway và Application Load Balancer.

### Lambda trigger của Cognito User Pools (Trang 729)

- Cognito User Pool có thể gọi Lambda function đồng bộ trong các trigger sau:

| Nhóm sự kiện | Trigger | Mục đích theo slide |
|---|---|---|
| Xác thực | Pre Authentication | Kiểm tra tùy chỉnh để chấp nhận hoặc từ chối yêu cầu đăng nhập. |
| Xác thực | Post Authentication | Ghi log sự kiện cho analytics tùy chỉnh. |
| Xác thực / tạo token | Pre Token Generation | Bổ sung hoặc loại bỏ claims của token; thêm hoặc bỏ thuộc tính trong ID token. |
| Đăng ký | Pre Sign-up | Kiểm tra tùy chỉnh để chấp nhận hoặc từ chối yêu cầu đăng ký. |
| Đăng ký | Post Confirmation | Tùy chỉnh welcome message hoặc ghi log cho analytics. |
| Đăng ký | Migrate User | Di chuyển người dùng từ user directory hiện có sang user pool. |
| Tin nhắn | Custom Message | Tùy chỉnh nâng cao và bản địa hóa nội dung tin nhắn. |

### Hosted authentication UI của Cognito User Pool (Trang 730-731)

- Cognito có hosted authentication UI để xử lý quy trình đăng ký và đăng nhập.
- Đây là nền tảng để tích hợp đăng nhập mạng xã hội, OIDC hoặc SAML.
- Có thể tùy chỉnh bằng logo và CSS.
- Với custom domain, phải tạo ACM certificate tại `us-east-1`.
- Khai báo custom domain trong mục **App Integration**.

### Application Load Balancer xác thực qua Cognito User Pool (Trang 732-733)

- Tạo Cognito User Pool, client và domain.
- Đảm bảo ID token được trả về.
- Thêm social hoặc corporate IdP nếu cần.
- Cấu hình các URL chuyển hướng cần thiết.
- Cho phép Cognito User Pool domain trong callback URL của ứng dụng IdP, ví dụ:
  - `https://domainprefix.auth.region.amazoncognito.com/saml2/idpresponse`
  - `https://user-pool-domain/oauth2/idpresponse`

### Cognito Identity Pools (Federated Identities) (Trang 734-736)

- Cấp danh tính cho người dùng để họ nhận temporary AWS credentials.
- Identity Pool có thể dùng các identity source:
  - Nhà cung cấp công khai như Login with Amazon, Facebook, Google và Apple.
  - Người dùng trong Cognito User Pool.
  - OpenID Connect Provider và SAML Identity Provider.
  - Developer Authenticated Identities từ máy chủ đăng nhập tùy chỉnh.
- Có thể cho phép truy cập không cần xác thực (guest).
- Người dùng có thể truy cập dịch vụ AWS trực tiếp hoặc qua API Gateway.
- IAM policy gắn với credentials được xác định trong Cognito; có thể tùy chỉnh policy theo `user_id` để kiểm soát chi tiết.

> *Sơ đồ (Trang 735-736): Identity Pool hoạt động độc lập và kết hợp với Cognito User Pool.*

### Cognito User Pools và Cognito Identity Pools (Trang 737)

- **Cognito User Pools — xác thực danh tính:**
  - Cơ sở dữ liệu người dùng cho ứng dụng web/mobile.
  - Liên kết đăng nhập với mạng xã hội, OIDC và SAML.
  - Có thể tùy chỉnh hosted UI.
  - Có Lambda trigger trong luồng xác thực.
  - Có thể điều chỉnh trải nghiệm đăng nhập theo mức rủi ro, ví dụ MFA hoặc adaptive authentication.
- **Cognito Identity Pools — cấp quyền truy cập AWS:**
  - Cấp AWS credentials cho người dùng.
  - Chấp nhận người dùng từ mạng xã hội, OIDC, SAML và Cognito User Pools.
  - Hỗ trợ guest.
  - Ánh xạ người dùng sang IAM Role và policy; có thể dùng policy variable.
- Kết hợp CUP và CIP để xây dựng luồng xác thực và cấp AWS credentials.

## Các dịch vụ serverless khác (Trang 738-760)

### AWS Step Functions (Trang 739)

- Mô hình hóa workflow thành state machine; mỗi workflow có một state machine.
- Dùng cho xử lý đơn hàng, xử lý dữ liệu, ứng dụng web và các workflow khác.
- Định nghĩa state machine bằng JSON.
- Cung cấp hình ảnh trực quan về workflow, quá trình thực thi và lịch sử execution.
- Có thể khởi chạy workflow bằng SDK, API Gateway hoặc EventBridge (trước đây là CloudWatch Events).

### Task State trong Step Functions (Trang 740)

- Task State thực hiện công việc trong state machine bằng cách gọi một AWS service, ví dụ:
  - Gọi Lambda function.
  - Chạy AWS Batch job.
  - Chạy ECS task và chờ task hoàn tất.
  - Thêm item vào DynamoDB.
  - Publish message lên SNS hoặc SQS.
  - Khởi chạy workflow Step Functions khác.
- Có thể chạy Activity trên EC2, ECS hoặc hạ tầng tại chỗ.
- Activity worker poll Step Functions để nhận công việc và gửi kết quả trở lại.

> *Sơ đồ (Trang 741): Ví dụ state machine gọi Lambda function.*

### Các loại state trong Step Functions (Trang 742)

- **Choice State:** kiểm tra điều kiện để chuyển sang nhánh tương ứng hoặc nhánh mặc định.
- **Fail / Succeed State:** kết thúc execution với trạng thái thất bại hoặc thành công.
- **Pass State:** chuyển input sang output hoặc thêm dữ liệu cố định mà không thực hiện công việc.
- **Wait State:** trì hoãn trong một khoảng thời gian hoặc đến thời điểm xác định.
- **Map State:** lặp động qua các bước.
- **Parallel State:** khởi chạy các nhánh thực thi song song.

> *Sơ đồ (Trang 743): Workflow trực quan trong Step Functions.*

### Xử lý lỗi trong Step Functions (Trang 744-747)

- State có thể gặp lỗi runtime do định nghĩa state machine, Task thất bại hoặc sự cố tạm thời như gián đoạn mạng.
- Dùng `Retry` để thử lại state lỗi và `Catch` để chuyển sang nhánh xử lý lỗi, thay vì tự triển khai toàn bộ luồng xử lý trong mã ứng dụng.
- Một số error name dựng sẵn:
  - `States.ALL`: khớp mọi tên lỗi.
  - `States.Timeout`: Task chạy lâu hơn `TimeoutSeconds` hoặc không nhận heartbeat.
  - `States.TaskFailed`: execution của Task thất bại.
  - `States.Permissions`: không đủ quyền thực thi.
- State cũng có thể báo lỗi riêng.
- Cấu hình `Retry` được đánh giá từ trên xuống; gồm:
  - `ErrorEquals`: loại lỗi cần khớp.
  - `IntervalSeconds`: thời gian chờ ban đầu.
  - `BackoffRate`: hệ số tăng thời gian chờ giữa các lần thử.
  - `MaxAttempts`: mặc định là 3; đặt 0 để không retry.
- Khi hết số lần thử, `Catch` được thực thi.
- Cấu hình `Catch` được đánh giá từ trên xuống; gồm `ErrorEquals`, `Next` (state tiếp theo) và `ResultPath` (vị trí đưa dữ liệu lỗi vào input của state tiếp theo).
- `ResultPath` có thể dùng để giữ lại input ban đầu và bổ sung thông tin lỗi.

### Chờ callback bằng Task Token (Trang 748-749)

- Task có thể tạm dừng cho đến khi nhận lại Task Token.
- Hữu ích khi chờ dịch vụ AWS khác, phê duyệt của con người, tích hợp bên thứ ba hoặc hệ thống cũ.
- Thêm `.waitForTaskToken` vào trường `Resource` để yêu cầu Step Functions chờ token.
- Task tiếp tục khi có lời gọi `SendTaskSuccess` hoặc `SendTaskFailure` kèm Task Token.
- Với Activity Task, Activity worker poll bằng `GetActivityTask`, sau đó gọi `SendTaskSuccess` hoặc `SendTaskFailure` để gửi kết quả.
- Đặt `TimeoutSeconds` cho thời gian tối đa chờ và định kỳ gửi heartbeat bằng `SendTaskHeartbeat` trong khoảng `HeartbeatSeconds`.
- Với `TimeoutSeconds` dài và heartbeat định kỳ, Activity Task có thể chờ tối đa một năm.

### Standard và Express workflow (Trang 751)

| Đặc điểm | Standard | Express |
|---|---|---|
| Thời lượng tối đa | Tối đa một năm. | Tối đa năm phút. |
| Mô hình thực thi theo slide | Exactly-once. | Asynchronous: at-least-once; synchronous: at-most-once. |
| Tốc độ execution theo slide | Trên 2.000/giây. | Trên 100.000/giây. |
| Lịch sử execution theo slide | Tối đa 90 ngày hoặc dùng CloudWatch. | CloudWatch Logs. |
| Cách tính phí theo slide | Số lần chuyển state. | Số execution, thời lượng và mức dùng bộ nhớ. |
| Ví dụ use case theo slide | Tác vụ không idempotent, ví dụ xử lý thanh toán. | Nhập dữ liệu IoT, streaming data, backend mobile. |

- Express asynchronous không chờ workflow hoàn tất; lấy kết quả từ CloudWatch Logs và phù hợp khi caller không cần response ngay, ví dụ luồng xử lý message.
- Với Express asynchronous, ứng dụng cần tự quản lý tính idempotent.
- Express synchronous chờ workflow hoàn tất, phù hợp khi cần response ngay hoặc điều phối microservice; có thể được gọi từ API Gateway hoặc Lambda.

### AWS AppSync (Trang 752-754)

- AppSync là dịch vụ managed dùng GraphQL.
- GraphQL giúp ứng dụng lấy đúng dữ liệu cần thiết và kết hợp dữ liệu từ một hoặc nhiều nguồn:
  - NoSQL datastore, cơ sở dữ liệu quan hệ và HTTP API.
  - Tích hợp DynamoDB, Aurora, OpenSearch và các nguồn khác.
  - Có thể dùng Lambda làm custom data source.
- Hỗ trợ lấy dữ liệu theo thời gian thực qua WebSocket hoặc MQTT trên WebSocket.
- Với ứng dụng mobile, hỗ trợ truy cập dữ liệu cục bộ và đồng bộ dữ liệu.
- Bắt đầu bằng cách tải lên GraphQL schema.

> *Sơ đồ (Trang 753): Ví dụ GraphQL và kiến trúc AppSync kết nối resolver với data source.*

- Có bốn cách xác thực ứng dụng với AppSync GraphQL API:
  - `API_KEY`
  - `AWS_IAM` — IAM user/role hoặc truy cập cross-account.
  - `OPENID_CONNECT` — OIDC provider/JWT.
  - `AMAZON_COGNITO_USER_POOLS`.
- Để dùng custom domain và HTTPS, đặt CloudFront phía trước AppSync.

### AWS Amplify (Trang 755-759)

- Bộ công cụ hỗ trợ bắt đầu xây dựng ứng dụng mobile và web; được slide ví như Elastic Beanstalk cho ứng dụng mobile/web.
- Cung cấp các khả năng như lưu trữ dữ liệu, xác thực, lưu trữ tệp và machine learning dựa trên các dịch vụ AWS.
- Có thư viện frontend và component dùng sẵn cho React.js, Vue, JavaScript, iOS, Android, Flutter và các nền tảng khác.
- Tích hợp best practice về độ tin cậy, bảo mật và khả năng mở rộng.
- Có thể build và triển khai bằng Amplify CLI hoặc Amplify Studio.

#### Một số tính năng của Amplify (Trang 757-758)

- **Authentication:** dùng Cognito; hỗ trợ đăng ký, xác thực, khôi phục tài khoản, MFA, social sign-in, component UI dựng sẵn và phân quyền chi tiết.
- **DataStore:** dùng AppSync và DynamoDB; làm việc với dữ liệu cục bộ và tự đồng bộ lên cloud; hỗ trợ GraphQL, offline và cập nhật theo thời gian thực; có mô hình dữ liệu trực quan trong Amplify Studio.

#### Amplify Hosting (Trang 759)

- Build và host ứng dụng web hiện đại.
- CI/CD gồm build, test và deploy.
- Có preview cho pull request và custom domain.
- Hỗ trợ monitoring, redirect, custom header và bảo vệ bằng password.

#### Kiểm thử end-to-end với Amplify (Trang 760)

- Chạy E2E test trong giai đoạn test của Amplify để phát hiện regression trước khi đưa mã lên production.
- Dùng bước test để chạy lệnh kiểm thử lúc build, cấu hình trong `amplify.yml`.
- Tích hợp Cypress và có thể tạo báo cáo UI cho kiểm thử.

### Bài tập trên slide (sau Trang 760)

- Tạo, build và triển khai ứng dụng Hello World mẫu bằng AWS SAM.
