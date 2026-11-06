# Section 16 - Advanced Identity, AWS Security & Encryption và các dịch vụ khác (Trang 800-847)

> *Section divider (Trang 800): Advanced Identity; AWS Security & Encryption gồm KMS, Encryption SDK, SSM Parameter Store, IAM và STS; cùng các AWS service khác.*

---

## Advanced Identity in AWS (Trang 801)

### AWS STS – Security Token Service (Trang 802)

- AWS STS cấp quyền truy cập giới hạn và tạm thời vào AWS resources; slide nêu thời hạn tối đa một giờ.
- **`AssumeRole`:** assume role trong cùng account hoặc account khác.
- **`AssumeRoleWithSAML`:** trả temporary credentials cho user đăng nhập qua SAML.
- **`AssumeRoleWithWebIdentity`:** trả credentials cho user đăng nhập bằng một IdP như Facebook Login, Google Login hoặc OIDC-compatible IdP; slide khuyến nghị dùng Cognito Identity Pools thay vì gọi trực tiếp API này.
- **`GetSessionToken`:** cấp session có MFA cho IAM user hoặc AWS account root user.
- **`GetFederationToken`:** lấy temporary credentials cho federated user.
- **`GetCallerIdentity`:** trả thông tin IAM user hoặc role đang dùng để gọi API.
- **`DecodeAuthorizationMessage`:** giải mã thông báo lỗi khi AWS API request bị từ chối.

### Dùng STS để assume role (Trang 803)

- Tạo IAM Role trong account của mình hoặc trong account khác.
- Khai báo principal nào được phép truy cập IAM Role đó.
- Dùng AWS STS `AssumeRole` để lấy credentials và hoạt động với quyền của IAM Role đã được cho phép.
- Theo slide, temporary credentials có thời hạn từ 15 phút đến 1 giờ.

> *Sơ đồ (Trang 803): Principal gọi STS `AssumeRole`, nhận temporary credentials rồi sử dụng quyền của role được assume.*

### Truy cập cross-account bằng STS (Trang 804)

- Tạo IAM Role trong account đích và cấu hình account/principal nguồn được phép assume role.
- Dùng AWS STS lấy temporary credentials để truy cập tài nguyên trong account đích.

> *Sơ đồ (Trang 804): Workload hoặc user ở một AWS account assume role trong account khác để truy cập tài nguyên được cấp quyền.*

### STS với MFA (Trang 805)

- Dùng `GetSessionToken` từ STS.
- Tạo IAM policy phù hợp, dùng condition `aws:MultiFactorAuthPresent: true`.
- Response của `GetSessionToken` gồm Access ID, Secret Key, Session Token và thời điểm hết hạn.

### IAM best practices – tổng quát (Trang 806)

- Không dùng Root Credentials; bật MFA cho Root Account.
- Cấp **least privilege**: mỗi group, user hoặc role chỉ có quyền tối thiểu cần thiết.
- Không cấp policy có quyền `*` cho một service.
- Dùng CloudTrail theo dõi các API call của user, đặc biệt các request bị từ chối.
- Slide khuyến cáo không lưu IAM key credentials trên máy ngoài personal computer hoặc on-premises server; với on-premises server, nên gọi STS để nhận temporary security credentials.

### IAM best practices – IAM Role (Trang 807)

- Mỗi EC2 machine nên có IAM Role riêng.
- Mỗi Lambda function nên có IAM Role riêng.
- ECS Task nên có IAM Role riêng; slide nêu cấu hình `ECS_ENABLE_TASK_IAM_ROLE=true`.
- CodeBuild nên có service role riêng.
- Tạo role có least privilege cho từng service cần quyền truy cập.
- Tạo role riêng theo application hoặc Lambda function; không dùng lại một role cho nhiều application/function.

### IAM best practices – cross-account access (Trang 808)

- Tạo IAM Role cho account khác truy cập.
- Khai báo account/principal được phép assume IAM Role.
- Dùng AWS STS `AssumeRole` để lấy temporary credentials và hoạt động với role.
- Theo slide, temporary credentials có thời hạn từ 15 phút đến 1 giờ.

### Mô hình đánh giá IAM policy (Trang 809)

1. Nếu có **explicit DENY**, kết quả là **DENY**.
2. Nếu có **ALLOW** và không có explicit DENY, kết quả là **ALLOW**.
3. Nếu không có policy cho phép, kết quả là **DENY**.

### IAM Policy và S3 Bucket Policy (Trang 810-814)

- IAM Policy gắn với user, role hoặc group; S3 Bucket Policy gắn với bucket.
- Khi đánh giá principal có thể thực hiện operation trên bucket hay không, slide mô tả việc xét kết hợp các IAM Policy được gán và S3 Bucket Policy.

| Ví dụ trên slide | IAM Role policy | S3 Bucket Policy | Kết quả |
|---|---|---|---|
| 1 (Trang 811) | Cho phép read/write trên `my_bucket` | Không có | EC2 instance có thể read/write bucket. |
| 2 (Trang 812) | Cho phép read/write trên `my_bucket` | Explicit deny role đó | EC2 instance không thể read/write bucket. |
| 3 (Trang 813) | Không có quyền S3 cho bucket | Explicit allow read/write role đó | EC2 instance có thể read/write bucket. |
| 4 (Trang 814) | Explicit deny S3 | Explicit allow role đó | EC2 instance không thể read/write bucket. |

### Dynamic policy với IAM (Trang 815-816)

- Ví dụ: cấp cho từng user một thư mục `/home/<user>` trong S3 bucket.
- Nếu tạo policy riêng cho từng user như `georges`, `sarah`, `matt`, số lượng policy không mở rộng tốt.
- Dùng một dynamic policy với policy variable `${aws:username}` để tạo đường dẫn theo username.

> *Sơ đồ (Trang 816): Policy variable `${aws:username}` tạo phạm vi S3 prefix tương ứng với từng user.*

### Inline Policy và managed policy (Trang 817)

| Loại policy | Đặc điểm theo slide |
|---|---|
| AWS Managed Policy | AWS duy trì; phù hợp power user/administrator; được cập nhật khi có service/API mới. |
| Customer Managed Policy | Best practice; có thể tái sử dụng cho nhiều principal; hỗ trợ version control, rollback và quản lý thay đổi tập trung. |
| Inline Policy | Quan hệ một-một với principal; policy bị xóa khi IAM principal bị xóa. |

### Microsoft Active Directory (AD) là gì? (Trang 818)

- Có trên Windows Server cài AD Domain Services.
- Cơ sở dữ liệu các object như user account, computer, printer, file share và security group.
- Quản lý bảo mật tập trung để tạo account và gán permission.
- Các object được tổ chức thành tree; một nhóm tree tạo thành forest.

### AWS Directory Services (Trang 819)

| Dịch vụ | Đặc điểm theo slide |
|---|---|
| AWS Managed Microsoft AD | Tạo AD riêng trên AWS, quản lý user tại đó, hỗ trợ MFA và thiết lập trust với AD on-premises. |
| AD Connector | Directory gateway/proxy chuyển request tới AD on-premises; hỗ trợ MFA; user vẫn được quản lý trên AD on-premises. |
| Simple AD | Directory tương thích AD được AWS quản lý; không thể join với AD on-premises. |

## AWS Security & Encryption: KMS, Encryption SDK, SSM Parameter Store (Trang 820-840)

### Vì sao cần encryption? – encryption in flight (Trang 821)

- Data được mã hóa trước khi gửi và giải mã sau khi nhận.
- TLS certificate hỗ trợ encryption cho HTTPS.
- Encryption in flight giúp ngăn tấn công man-in-the-middle (MITM).

### Vì sao cần encryption? – server-side encryption at rest (Trang 822)

- Server mã hóa data sau khi nhận và giải mã trước khi gửi đi.
- Data được lưu dưới dạng mã hóa bằng key, thường là data key.
- Key mã hóa/giải mã phải được quản lý; server cần quyền truy cập key.

### Vì sao cần encryption? – client-side encryption (Trang 823)

- Client mã hóa data; server không giải mã được.
- Client nhận dữ liệu chịu trách nhiệm giải mã.
- Có thể dùng Envelope Encryption.

### AWS KMS – Key Management Service (Trang 824)

- Khi AWS service có yêu cầu encryption, KMS thường là dịch vụ quản lý key liên quan.
- AWS quản lý encryption key; KMS tích hợp với IAM để kiểm soát quyền truy cập data.
- Dùng CloudTrail để audit việc sử dụng KMS Key.
- Tích hợp với nhiều AWS service như EBS, S3, RDS và SSM.
- Không lưu secret ở dạng plaintext, đặc biệt trong source code.
- Có thể dùng KMS API qua SDK hoặc CLI để encrypt/decrypt; slide nêu encrypted secret có thể lưu trong code hoặc environment variable.

### Các loại KMS Key (Trang 825)

- Tên **KMS Key** thay cho tên cũ **Customer Master Key (CMK)**.
- **Symmetric (AES-256):** một key dùng cho cả Encrypt và Decrypt; AWS service tích hợp KMS sử dụng symmetric key; không lấy key ở dạng plaintext mà phải gọi KMS API.
- **Asymmetric (RSA và ECC):** cặp public/private key dùng cho Encrypt/Decrypt hoặc Sign/Verify; public key có thể tải xuống, private key không thể lấy ở dạng plaintext.
- Use case của asymmetric key: encryption bên ngoài AWS khi người dùng không thể gọi KMS API.

### KMS Key types, pricing và rotation (Trang 826)

| Loại key | Đặc điểm/giá theo slide |
|---|---|
| AWS Owned Keys | Miễn phí; ví dụ SSE-S3, SSE-SQS, SSE-DDB (default key). |
| AWS Managed Key | Miễn phí; tên theo dạng `aws/service-name`, ví dụ `aws/rds`, `aws/ebs`. |
| Customer managed key tạo trong KMS | $1/tháng. |
| Customer managed key dùng imported key material | $1/tháng; imported key phải là symmetric key. |
| KMS API request | Cộng thêm $0.03/10.000 request. |

- **Automatic key rotation theo slide:** AWS Managed KMS Key tự động mỗi năm; Customer Managed KMS Key cần bật rotation và tự động mỗi năm; imported KMS Key chỉ rotation thủ công bằng alias.

### Copy snapshot qua Region (Trang 827)

- EBS Volume và snapshot tại `eu-west-2` được mã hóa bằng KMS Key A.
- Khi copy snapshot sang `ap-southeast-2`, KMS re-encrypt snapshot bằng KMS Key B tại Region đích.

> *Sơ đồ (Trang 827): Snapshot mã hóa bằng Key A tại Region nguồn được copy và mã hóa lại bằng Key B tại Region đích.*

### KMS Key Policy (Trang 828)

- Key Policy kiểm soát quyền truy cập KMS Key, tương tự S3 Bucket Policy.
- Không thể kiểm soát truy cập KMS Key nếu không có Key Policy.
- Nếu không cung cấp policy riêng, KMS tạo default key policy; root user có toàn quyền với key trong account.
- Custom Key Policy xác định user/role được dùng key và principal được quản trị key; hữu ích cho cross-account access.

### Copy snapshot cross-account (Trang 829)

1. Tạo snapshot được mã hóa bằng Customer Managed KMS Key của account nguồn.
2. Gắn KMS Key Policy cho phép account khác truy cập key.
3. Share snapshot đã mã hóa.
4. Trong account đích, copy snapshot và mã hóa bằng Customer Managed KMS Key của account đích.
5. Tạo volume từ snapshot đã copy.

### KMS Encrypt và Decrypt API (Trang 830)

- Client gửi secret có kích thước dưới 4 KB tới KMS Encrypt API.
- KMS kiểm tra quyền IAM và thực hiện encryption bằng KMS Key.
- KMS trả encrypted data; khi cần, client gọi Decrypt API và KMS kiểm tra quyền trước khi trả plaintext.

> *Sơ đồ (Trang 830): Encrypt/Decrypt API gọi KMS; KMS kiểm tra IAM permissions trước khi dùng key và trả encrypted/decrypted data.*

### Envelope Encryption (Trang 831-832)

- KMS Encrypt API giới hạn dữ liệu tối đa 4 KB.
- Muốn encrypt data lớn hơn 4 KB, dùng Envelope Encryption với `GenerateDataKey`.
- KMS trả plaintext data key để mã hóa data và encrypted data key để lưu cùng ciphertext.

> *Sơ đồ (Trang 832): Client dùng plaintext data key mã hóa file lớn; lưu encrypted file cùng encrypted data key. KMS bảo vệ data key bằng KMS Key.*

### AWS CloudHSM (Trang 833)

- Với KMS, AWS quản lý phần mềm encryption; với CloudHSM, AWS cung cấp encryption hardware.
- CloudHSM dùng Hardware Security Module chuyên dụng; khách hàng tự quản lý encryption key.
- HSM chống can thiệp vật lý, tuân thủ FIPS 140-2 Level 3 theo slide.
- Hỗ trợ symmetric/asymmetric encryption và SSL/TLS key.
- Không có free tier; cần CloudHSM Client Software.
- Redshift hỗ trợ CloudHSM cho database encryption và key management; slide nêu CloudHSM là lựa chọn phù hợp với SSE-C.

### SSM Parameter Store (Trang 834-836)

- Lưu trữ cấu hình và secret an toàn; có thể bật encryption bằng KMS.
- Serverless, có khả năng mở rộng, bền vững, hỗ trợ SDK.
- Theo dõi version của cấu hình/secret; bảo vệ bằng IAM.
- Có thể gửi notification qua EventBridge và tích hợp CloudFormation.

**Parameter hierarchy (Trang 835):**

- Ví dụ cấu trúc: `/my-department/my-app/dev/db-url`, `/my-department/my-app/prod/db-password`.
- Có thể tham chiếu Secrets Manager qua `/aws/reference/secretsmanager/secret_ID_in_Secrets_Manager`.
- Có thể tham chiếu public AMI như `/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2`.

**Standard và Advanced tier (Trang 836):**

| Thuộc tính theo slide | Standard | Advanced |
|---|---:|---:|
| Tổng số parameter/account/Region | 10.000 | 100.000 |
| Kích thước tối đa value | 4 KB | 8 KB |
| Parameter policies | Không | Có |
| Storage | Miễn phí | Tính phí |
| Giá lưu trữ nêu trên slide | Miễn phí | $0.05/parameter/tháng |

### AWS Secrets Manager (Trang 837)

- Dịch vụ chuyên lưu trữ secret.
- Có thể cấu hình rotation sau mỗi khoảng thời gian.
- Có thể tự động tạo secret mới khi rotation bằng Lambda.
- Tích hợp với Amazon RDS for MySQL, PostgreSQL và Aurora.
- Secret được mã hóa bằng KMS.
- Slide nhấn mạnh use case tích hợp RDS.

### Secrets Manager Multi-Region Secret (Trang 838)

- Replicate secret qua nhiều AWS Region.
- Secrets Manager giữ read replica đồng bộ với primary secret.
- Có thể promote read replica thành standalone secret.
- Use case: ứng dụng multi-Region, disaster recovery và database đa Region.

### SSM Parameter Store và Secrets Manager (Trang 839-840)

| Đặc điểm theo slide | Secrets Manager | SSM Parameter Store |
|---|---|---|
| Rotation | Tự động rotation bằng Lambda; có Lambda function mẫu cho RDS, Redshift và DocumentDB. | Không có built-in secret rotation; có thể tự triển khai rotation bằng Lambda được EventBridge kích hoạt. |
| KMS | Encryption bằng KMS là bắt buộc. | KMS encryption là tùy chọn. |
| Tích hợp | Có thể tích hợp CloudFormation. | Có thể tích hợp CloudFormation; SSM API có thể lấy secret từ Secrets Manager. |
| Chi phí tương đối theo slide | Cao hơn (`$$$`). | Thấp hơn (`$`). |

> *Sơ đồ (Trang 840): Secrets Manager định kỳ gọi Lambda để đổi password RDS; với Parameter Store, EventBridge định kỳ gọi Lambda, đổi password rồi cập nhật parameter.*

## AWS Other Services (Trang 841-847)

### Amazon Simple Email Service (Amazon SES) (Trang 842)

- Dịch vụ fully managed để gửi email an toàn, toàn cầu và ở quy mô lớn; hỗ trợ inbound/outbound email.
- Có reputation dashboard, performance insights và anti-spam feedback.
- Cung cấp thống kê như email delivery, bounce, feedback loop và email open.
- Hỗ trợ DomainKeys Identified Mail (DKIM) và Sender Policy Framework (SPF).
- Hỗ trợ shared, dedicated và customer-owned IP.
- Ứng dụng có thể gửi email bằng AWS Console, API hoặc SMTP.
- Use case: transactional email, marketing và bulk email.

### Amazon OpenSearch Service (Trang 843)

- OpenSearch là dịch vụ kế tiếp Amazon Elasticsearch Service.
- Theo slide, DynamoDB query chủ yếu theo primary key hoặc index; OpenSearch hỗ trợ tìm kiếm trên field bất kỳ, kể cả partial match.
- Thường dùng OpenSearch bổ sung cho database khác.
- Có managed cluster và serverless cluster.
- Theo slide, SQL không có sẵn và có thể bật qua plugin.
- Có thể ingest dữ liệu từ Kinesis Data Firehose, AWS IoT và CloudWatch Logs.
- Bảo mật bằng Cognito/IAM, KMS encryption và TLS; có OpenSearch Dashboards để visualization.

### Amazon Athena (Trang 844)

- Serverless query service phân tích dữ liệu lưu trên S3.
- Dùng SQL tiêu chuẩn để query file; được xây dựng trên Presto.
- Hỗ trợ CSV, JSON, ORC, Avro và Parquet.
- Theo slide, giá là $5.00 mỗi TB dữ liệu được scan.
- Thường dùng cùng Amazon QuickSight cho report/dashboard.
- Use case: business intelligence, analytics, reporting và phân tích VPC Flow Logs, ELB Logs, CloudTrail trails.
- Exam tip trên slide: cần phân tích dữ liệu trong S3 bằng serverless SQL thì dùng Athena.

### Amazon Athena Federated Query (Trang 845)

- Chạy SQL query trên dữ liệu ở relational, non-relational, object và custom data source trong AWS hoặc on-premises.
- Data Source Connector chạy trên AWS Lambda để truy vấn nguồn như CloudWatch Logs, DynamoDB hoặc RDS.
- Kết quả được lưu lại trên S3.

### AWS Certificate Manager (ACM) (Trang 846)

- Cấp, quản lý và triển khai SSL/TLS Certificate.
- Dùng certificate để cung cấp encryption in flight cho website HTTPS.
- Hỗ trợ public và private TLS certificate.
- Public TLS certificate miễn phí theo slide; ACM tự động renew certificate.
- Có thể gắn TLS certificate lên Elastic Load Balancer, CloudFront Distribution và API Gateway API.

### Amazon Macie (Trang 847)

- Dịch vụ fully managed về data security và data privacy.
- Dùng Machine Learning và pattern matching để khám phá, bảo vệ sensitive data trên AWS.
- Giúp xác định và cảnh báo về dữ liệu nhạy cảm như Personally Identifiable Information (PII).
