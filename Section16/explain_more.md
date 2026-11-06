# Giải thích bổ sung - Section 16: Identity, Security và AWS services

> *Phần này bổ sung cách lựa chọn và vận hành dịch vụ; nội dung thực tế được tách khỏi bản dịch slide. Giá, giới hạn và trạng thái dịch vụ trong slide là dữ liệu theo khóa học; kiểm tra AWS documentation trước khi áp dụng.*

---

### 1. STS, AssumeRole và temporary credentials

#### Giải thích cốt lõi

AWS STS phát hành temporary credentials gồm access key ID, secret access key và session token. `AssumeRole` chỉ cấp session nếu role trust policy cho phép principal gọi role; role permissions policy giới hạn principal đó làm được gì sau khi assume. So với long-term access key gắn vào IAM user, role session tự hết hạn, có thể giới hạn thời gian và dễ audit hơn. `GetSessionToken` dùng credentials của IAM user để tạo session, thường nhằm đưa MFA context vào các API request; nó không giống `AssumeRole`, vốn đổi sang quyền của role. Với workload trên EC2/Lambda/ECS, dùng execution role/task role thay vì lưu access key trong file.

Slide nêu `AssumeRole` 15 phút–1 giờ. Tài liệu AWS hiện tại cho phép role session tối đa theo setting của role (1–12 giờ), nhưng role chaining bị giới hạn tối đa một giờ. `GetSessionToken` có giới hạn khác; không nên dùng phiên root cho hoạt động thường ngày. Nguồn: [AssumeRole API](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html), [GetSessionToken API](https://docs.aws.amazon.com/STS/latest/APIReference/API_GetSessionToken.html).

#### NÊN làm (DO)

- **Dùng IAM Role và temporary credentials cho workload hoặc truy cập cross-account** → Credentials tự hết hạn và permission được gắn với role; so với access key dài hạn được chép giữa máy, giảm rủi ro lộ và đơn giản hóa thu hồi.
- **Tách trust policy khỏi permissions policy** → Trust trả lời “ai được assume”, permissions trả lời “role làm gì”; so với một policy rộng khó kiểm tra, giúp khoanh vùng lỗi và review rõ hơn.
- **Yêu cầu MFA trong trust policy/API path khi truy cập đặc quyền cần MFA** → STS có thể kiểm tra MFA context; so với chỉ bật MFA đăng nhập mà không ràng buộc API access, đảm bảo điều kiện bảo vệ được áp dụng tại thao tác cần bảo vệ.
- **Ghi role session/source identity có thể truy vết và đặt thời hạn vừa đủ** → CloudTrail có thể gắn thao tác với session; so với session dài, khó quy nguồn, giới hạn thời gian giảm khoảng credential bị lạm dụng.

#### KHÔNG NÊN làm (DON'T)

- **Lưu/commit long-term access key hoặc dùng chung IAM user key cho ứng dụng** → Key bị lộ có thể bị dùng lâu dài; câu trên slide không nên hiểu là khuyến nghị cất long-term key trên laptop/on-premises. NÊN dùng federation/temporary role credentials hoặc service role.
- **Cho rằng `GetSessionToken` làm giảm quyền của IAM user** → Credentials nhận được thường giữ permission của IAM user; NÊN dùng role/session policy nếu cần chuyển hoặc giới hạn quyền.
- **Dùng Root Credentials để gọi API hoặc thử MFA flow** → Session root vẫn có quyền root; NÊN khóa root cho tác vụ cần thiết, bật MFA và dùng IAM identity riêng.

---

### 2. IAM policy evaluation, S3 và policy động

#### Giải thích cốt lõi

Quy tắc an toàn để đọc policy là explicit deny thắng, không có allow thì implicit deny. Tuy nhiên “union” trong slide không phải quy tắc duy nhất cho mọi loại policy: permissions boundary, session policy, Service Control Policy (SCP), resource policy và loại principal đều ảnh hưởng kết quả. Đặc biệt, cross-account request cần cả account của principal và account sở hữu resource cho phép. Ví dụ S3: account nguồn phải cho phép action trong identity policy; bucket policy ở account đích cũng phải cho principal đó truy cập. Nguồn: [IAM policy evaluation](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html), [cross-account evaluation](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic-cross-account.html).

`${aws:username}` phù hợp với policy theo IAM user, như giới hạn mỗi user vào prefix S3 riêng. Nó không phải định danh chung cho mọi principal, chẳng hạn assumed role không có username theo cách đó; workload dùng role thường cần thiết kế resource scope hoặc principal/session tags thay thế. AWS Managed Policy tiện cho quyền phổ biến nhưng AWS có thể thay đổi nội dung; Customer Managed Policy cho phép chủ động review/version; Inline Policy gắn chặt với một principal.

#### NÊN làm (DO)

- **Bắt đầu bằng resource/action cụ thể và kiểm tra các lớp giới hạn cùng lúc** → Boundary/SCP/session policy có thể làm mất quyền dù identity policy có Allow; so với sửa policy bằng cách thêm `*`, tìm đúng lớp đang deny an toàn hơn.
- **Trong cross-account, kiểm tra cả principal-side IAM policy và resource-side policy/trust policy** → Cả hai account phải có quyết định Allow theo luồng phù hợp; so với chỉ thêm bucket policy, tránh request vẫn bị từ chối.
- **Dùng Customer Managed Policy cho quyền riêng của ứng dụng cần review và version control** → Thay đổi được kiểm soát và tái sử dụng; so với chỉnh trực tiếp một loạt user policy, dễ audit hơn.
- **Dùng policy variable chỉ khi principal context có biến đó; với role cân nhắc principal tag/ABAC** → Scope khớp danh tính thực tế; so với giả định `${aws:username}` tồn tại cho mọi principal, tránh policy không hoạt động hoặc giới hạn sai.

#### KHÔNG NÊN làm (DON'T)

- **Đọc “union” thành “chỉ cần một Allow là mọi request được phép”** → Cross-account và permission boundary có phép kiểm tra riêng; NÊN lần theo policy evaluation của cả account và các giới hạn áp dụng.
- **Cho quyền `Action: "*"` hoặc Resource `"*"` để chữa lỗi AccessDenied** → Blast radius tăng và explicit deny/condition khác vẫn có thể chặn; NÊN dùng CloudTrail, decoded authorization message khi hỗ trợ và policy simulator để xác định action/resource thiếu.
- **Tạo hàng trăm policy tĩnh theo từng user hoặc dùng Inline Policy làm mặc định** → Khó quản trị thay đổi và dễ lệch permission; NÊN dùng policy tái sử dụng hoặc policy variable/tag có scope rõ.

---

### 3. Chọn AWS Directory Service

#### Giải thích cốt lõi

AWS Managed Microsoft AD chạy Microsoft Active Directory do AWS quản lý; phù hợp khi ứng dụng cần AD thật, trust, Group Policy hoặc quản lý user trong directory AWS. AD Connector là proxy tới AD tự quản lý, không phải nơi tạo bản sao user directory; dùng khi danh tính vẫn phải nằm trên AD hiện hữu. Simple AD là directory tương thích AD dựa trên Samba, có tập tính năng khác Managed Microsoft AD. Slide nói Simple AD không join với on-premises AD; AWS hiện không nhận onboarding khách hàng mới cho Simple AD nhưng khách hàng hiện hữu vẫn dùng được. Nguồn: [Simple AD availability changes](https://docs.aws.amazon.com/directoryservice/latest/admin-guide/simple-ad-availability-change.html).

#### NÊN làm (DO)

- **Chọn Managed Microsoft AD khi cần AD native do AWS vận hành hoặc trust với AD hiện hữu** → Có thể dùng các tính năng AD phù hợp hơn; so với Simple AD, không nhầm directory tương thích với Microsoft AD đầy đủ.
- **Chọn AD Connector khi cần proxy xác thực tới directory tự quản lý** → User/password policy vẫn nằm trong AD nguồn; so với migrate directory, tránh đồng bộ/copy danh tính sang một directory khác.
- **Xác nhận availability và yêu cầu trust/MFA trước khi chọn loại directory** → Các loại dịch vụ có khả năng khác nhau và trạng thái onboarding có thể đổi; so với dựng kiến trúc theo slide cũ, kiểm tra dịch vụ/Region hiện hành.

#### KHÔNG NÊN làm (DON'T)

- **Coi AD Connector là một directory chứa bản sao user** → Connector proxy tới AD nguồn; NÊN bảo đảm mạng, DNS và AD on-premises luôn truy cập được.
- **Chọn Simple AD cho hệ thống mới mà không kiểm tra onboarding** → AWS không mở Simple AD cho khách hàng mới theo thông báo hiện hành; NÊN đánh giá Managed Microsoft AD hoặc AD Connector.

---

### 4. Chọn lớp encryption: in transit, at rest và client-side

#### Giải thích cốt lõi

TLS bảo vệ dữ liệu trên đường truyền giữa client và endpoint; server-side encryption at rest bảo vệ dữ liệu lưu ở dịch vụ bằng key mà dịch vụ được phép dùng. Client-side encryption mã hóa trước khi gửi và giữ khả năng giải mã ở client; server không thể đọc plaintext nếu không có key. Ba lớp giải quyết các threat khác nhau, không thay thế lẫn nhau. Ví dụ HTTPS bảo vệ request khi truyền nhưng không tự quyết định ai đọc object sau khi lưu; SSE at rest bảo vệ storage nhưng dịch vụ có thể giải mã khi phục vụ request được cấp quyền.

#### NÊN làm (DO)

- **Dùng HTTPS/TLS cho dữ liệu nhạy cảm trên mạng** → Ngăn đọc/sửa dữ liệu trên đường truyền; so với chỉ bật at-rest encryption, bảo vệ thêm hop giữa client và service.
- **Chọn SSE/KMS khi service cần xử lý plaintext nhưng storage phải mã hóa** → Service giải mã trong lúc xử lý theo quyền; so với client-side, vận hành tích hợp hơn nhưng trust service/key policy.
- **Dùng client-side/envelope encryption khi server không được phép đọc nội dung** → Plaintext chỉ xuất hiện tại client được ủy quyền; so với SSE, giảm mức tin cậy vào dịch vụ nhưng tăng trách nhiệm quản lý key và decrypt ở client.

#### KHÔNG NÊN làm (DON'T)

- **Xem biểu tượng HTTPS hoặc KMS là chứng minh mọi data đã được bảo vệ** → Có thể còn log, cache, backup hoặc plaintext ở client; NÊN lập data flow và chỉ ra từng nơi plaintext/key xuất hiện.
- **Tự thiết kế thuật toán hoặc ghi key vào cạnh ciphertext không bảo vệ** → Kẻ lấy được data và key có thể giải mã; NÊN dùng KMS/Encryption SDK hoặc một protocol đã được kiểm chứng.

---

### 5. KMS Key, Key Policy và encrypted snapshot

#### Giải thích cốt lõi

KMS Key là logical key dùng để gọi cryptographic operation; symmetric key material không được tải xuống plaintext. AWS service thường gọi KMS theo quyền của service/principal. Key Policy là gốc của access control cho customer-managed KMS key; IAM policy chỉ có hiệu lực khi key policy cho phép IAM policy ủy quyền, hoặc key policy cấp quyền trực tiếp. Cross-account cần cho phép ở key policy của account sở hữu key và IAM policy của principal bên kia. Rotation thay key material được dùng cho operation mới nhưng KMS vẫn giữ material cũ để giải mã ciphertext đã tạo trước đó. Giá, thuật ngữ key type và rotation interval trong slide có thể thay đổi, nên xem pricing/quota hiện hành.

Khi copy encrypted EBS snapshot sang Region khác, snapshot phải được re-encrypt bằng KMS key dùng được ở Region đích; khi share cross-account, phải cấp quyền cho cả snapshot và KMS key, rồi account đích copy sang key của mình. Tham khảo [KMS Key Policies](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html) và [cross-account key access](https://docs.aws.amazon.com/kms/latest/developerguide/key-policy-modifying-external-accounts.html).

#### NÊN làm (DO)

- **Tạo Customer Managed Key khi cần tự quản trị policy, rotation, alias hoặc audit theo ứng dụng** → Ranh giới quản trị rõ; so với AWS Managed Key, linh hoạt hơn nhưng cần owner và lịch review.
- **Giới hạn `kms:Decrypt`, `kms:Encrypt` và key administration riêng biệt** → Người có quyền dùng key không nhất thiết được sửa/xóa policy; so với một role có quyền toàn diện, giảm nguy cơ mất key hoặc lộ plaintext.
- **Dùng key riêng ở Region đích khi copy snapshot; kiểm tra cả snapshot permission và key permission khi share account** → Ciphertext được mở khóa đúng nơi với chính sách đích; so với copy nguyên key ARN từ Region nguồn, tránh key không khả dụng/không được phép.
- **Dùng CloudTrail và Encryption Context khi cần audit/ràng buộc operation theo ứng dụng** → Có thông tin truy vết/điều kiện bổ sung; so với dựa vào alias hoặc tên file, kiểm soát theo request rõ hơn.

#### KHÔNG NÊN làm (DON'T)

- **Cho rằng IAM Allow luôn đủ để dùng Customer Managed KMS Key** → Key policy có thể không ủy quyền cho IAM principal; NÊN kiểm tra key policy, IAM policy, grants và SCP/permission boundary.
- **Xóa hoặc disable key mà không đánh giá dữ liệu đã mã hóa bằng key đó** → Data có thể không giải mã được; NÊN kiểm tra phụ thuộc, retention và recovery trước khi schedule deletion.
- **Hard-code giá KMS/rotation như một thông số lâu dài** → Giá và tính năng có thể đổi theo Region/key type; NÊN đối chiếu AWS KMS pricing và documentation hiện hành trước khi lập dự toán.

---

### 6. Envelope Encryption và CloudHSM

#### Giải thích cốt lõi

KMS `Encrypt` API chỉ nhận plaintext tối đa 4 KB theo mô hình trong slide. Với file lớn, `GenerateDataKey` tạo plaintext data key và encrypted copy của data key. Ứng dụng dùng plaintext data key để mã hóa payload; lưu ciphertext cùng encrypted data key. Khi giải mã, KMS decrypt data key trước, ứng dụng giải mã payload. KMS không thực hiện encryption cho payload bằng data key thay ứng dụng; AWS Encryption SDK giúp đóng gói envelope format và key provider an toàn hơn. Nguồn: [Generate data keys](https://docs.aws.amazon.com/kms/latest/developerguide/data-keys.html).

CloudHSM cấp HSM chuyên dụng và khách hàng kiểm soát key material/hoạt động crypto ở mức khác KMS; đổi lại khách hàng chịu thêm trách nhiệm vận hành cluster, client, HA và phục hồi. Chọn CloudHSM theo yêu cầu kiểm soát/phần cứng, không chỉ vì “mã hóa mạnh hơn”.

#### NÊN làm (DO)

- **Dùng Envelope Encryption cho payload lớn và bỏ plaintext data key khỏi memory ngay sau khi dùng** → KMS chỉ bảo vệ data key, crypto payload chạy gần data; so với gửi cả file lớn qua KMS Encrypt, phù hợp giới hạn API và giảm lượng plaintext key tồn tại.
- **Dùng AWS Encryption SDK nếu cần format envelope, key commitment và mã hóa đa key provider** → SDK xử lý metadata/crypto workflow đã chuẩn hóa; so với tự ghép ciphertext và encrypted data key, giảm lỗi protocol.
- **Chọn CloudHSM khi chính sách yêu cầu dedicated HSM hoặc quyền kiểm soát key material cụ thể** → Có thể đáp ứng mô hình quản trị khác KMS; so với KMS managed, cần đánh đổi chi phí và vận hành.

#### KHÔNG NÊN làm (DON'T)

- **Giữ plaintext data key lâu hoặc ghi nó vào log/file tạm** → Ai lấy được key có thể giải mã payload; NÊN giới hạn memory lifetime và xóa key ngay khi xong.
- **Tự tuyên bố CloudHSM thay thế KMS cho mọi AWS service** → Tích hợp và thao tác khác nhau; NÊN xác nhận service/key requirement cụ thể trước khi thiết kế.

---

### 7. SSM Parameter Store và Secrets Manager

#### Giải thích cốt lõi

Parameter Store phù hợp với cấu hình có version, hierarchy và `SecureString` được mã hóa tùy chọn; Standard tier có giới hạn nhỏ hơn/ít tính năng hơn Advanced tier. Secrets Manager tập trung vào secret lifecycle như rotation, tích hợp rotation Lambda và replication secret đa Region. Có thể đọc Secrets Manager secret qua Parameter Store integration/API theo slide, nhưng không biến Parameter Store thành cùng một rotation engine. Khóa KMS, IAM policy, network access và secret consumer đều quyết định bảo mật thực tế.

Giới hạn 10.000/100.000 parameter, 4/8 KB và giá trong slide là con số khóa học; đối chiếu [Parameter Store tiers](https://docs.aws.amazon.com/systems-manager/latest/userguide/parameter-store-advanced-parameters.html) và pricing hiện tại. AWS cũng khuyến nghị Secrets Manager cho nhiều loại secret như database credential; xem [Parameter Store overview](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html).

#### NÊN làm (DO)

- **Dùng Parameter Store cho cấu hình phân cấp/versioned và SecureString đơn giản** → API gọn, có thể dùng tier phù hợp; so với tạo secret lifecycle phức tạp, ít thành phần hơn.
- **Dùng Secrets Manager khi cần managed secret rotation hoặc multi-Region replica** → Rotation/replication là khả năng trọng tâm; so với tự dựng EventBridge, Lambda, password update và replica sync, giảm code lifecycle tự duy trì.
- **Cho service role quyền đọc đúng parameter/secret path, không truyền secret vào build log** → Consumer nhận đúng secret theo runtime; so với hard-code secret hoặc environment/config mở rộng, giảm đường lộ dữ liệu.
- **Kiểm tra cost và quota theo tier/Region trước khi lưu nhiều parameter** → Advanced tier tính phí và quota khác; so với giả định toàn bộ Parameter Store miễn phí, tránh chi phí hoặc quota bất ngờ.

#### KHÔNG NÊN làm (DON'T)

- **Dùng Standard parameter cho value vượt giới hạn hoặc cần parameter policy** → Tạo/đọc có thể thất bại hoặc thiếu tính năng; NÊN chọn Advanced sau khi tính quota/cost.
- **Tự viết rotation mà không phối hợp đổi credential phía database và consumer** → Secret mới và password DB có thể lệch nhau gây outage; NÊN dùng managed rotation template hoặc workflow có kiểm thử, rollback và lịch phối hợp.
- **Coi việc bật KMS encryption là đủ để bảo vệ secret** → User có `GetParameter`/`GetSecretValue` vẫn có thể lấy plaintext; NÊN giới hạn IAM, log/audit và quyền decrypt.

---

### 8. Amazon SES và khả năng gửi email

#### Giải thích cốt lõi

SES gửi và nhận email qua API hoặc SMTP. Domain/email identity cần được xác minh; DKIM/SPF giúp xác thực sender domain và giảm giả mạo. Reputation, bounce, complaint và delivery feedback cho biết chất lượng danh sách/luồng gửi. So với tự vận hành mail server, SES giảm vận hành hạ tầng; nhưng ứng dụng vẫn chịu trách nhiệm consent, bounce handling, template và giới hạn sending.

Nguồn: [Configuring identities in SES](https://docs.aws.amazon.com/ses/latest/dg/configure-identities.html), [SPF authentication](https://docs.aws.amazon.com/ses/latest/dg/send-email-authentication-spf.html).

#### NÊN làm (DO)

- **Xác minh domain và cấu hình DKIM/SPF trước khi gửi production** → Receiver có thông tin xác thực sender; so với gửi không xác thực, giảm khả năng bị giả mạo hoặc vào spam.
- **Theo dõi bounce/complaint/reputation và loại địa chỉ không hợp lệ khỏi danh sách** → Tỷ lệ lỗi phản ánh chất lượng nguồn gửi; so với chỉ đếm số email gửi, phát hiện sớm vấn đề deliverability.
- **Dùng SES API/SMTP theo integration của ứng dụng và giới hạn IP/quyền gửi cần thiết** → Credential gửi email có phạm vi nhỏ; so với chia sẻ SMTP credential rộng, giảm tác động khi bị lộ.

#### KHÔNG NÊN làm (DON'T)

- **Gửi bulk email tới danh sách không có consent hoặc bỏ qua bounce/complaint** → Reputation/domain có thể xấu đi và email không tới inbox; NÊN quản lý opt-in, suppression và feedback loop.
- **Nhúng SMTP password hoặc AWS access key dài hạn vào source/frontend** → Credential có thể bị trích xuất; NÊN dùng backend role và secret store với quyền tối thiểu.

---

### 9. Amazon OpenSearch Service

#### Giải thích cốt lõi

OpenSearch đánh index dữ liệu để hỗ trợ search và phân tích; thường bổ sung cho database giao dịch như DynamoDB thay vì thay primary store. Tìm text/field/partial match có thể phù hợp với search index, trong khi primary database giữ record chuẩn. Slide nói SQL cần plugin; tài liệu OpenSearch Service hiện tại có SQL plugin cho các domain/version tương thích, nên xác minh engine/version thay vì ghi nhớ rằng dịch vụ không hỗ trợ SQL. Nguồn: [OpenSearch SQL support](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/sql-support.html).

#### NÊN làm (DO)

- **Dùng OpenSearch cho full-text/search/analytics cần lọc theo nhiều field** → Index được tối ưu cho truy vấn tìm kiếm; so với quét primary database cho mọi search query, tách tải search khỏi transaction path.
- **Giữ database giao dịch làm source of truth và đồng bộ index có thể tái tạo** → Search index phục vụ retrieval nhưng có thể rebuild; so với ghi business state chỉ vào search cluster, bảo vệ tính nhất quán và phục hồi.
- **Chọn managed cluster hay serverless theo pattern workload, scaling và vận hành** → Hai mode có trade-off capacity/control; so với chọn theo tên, đối chiếu traffic và feature cần dùng.

#### KHÔNG NÊN làm (DON'T)

- **Coi OpenSearch index luôn đồng bộ tức thời với database** → Pipeline ingest/update có thể trễ hoặc lỗi; NÊN thiết kế eventual consistency, retry và rebuild/reconciliation.
- **Mở domain hoặc query endpoint rộng để “dễ tích hợp”** → Có thể lộ dữ liệu và cho phép query tốn tài nguyên; NÊN dùng network/access policy, IAM/fine-grained access và TLS.
- **Dựa vào mô tả SQL trong slide mà không kiểm tra engine version** → Hỗ trợ SQL phụ thuộc plugin/version; NÊN xác minh endpoint, syntax và giới hạn của domain hiện tại.

---

### 10. Amazon Athena và Federated Query

#### Giải thích cốt lõi

Athena chạy SQL serverless trên dữ liệu lưu ở S3; chi phí query thường liên quan lượng dữ liệu scan, nên định dạng columnar, compression và partition pruning giúp giảm đọc không cần thiết. Federated Query dùng connector để query data source khác; connector chạy bằng Lambda, có quyền/network/dependency riêng và query result vẫn có thể lưu ở S3. So với load data trước vào warehouse, federated query có thể query in place nhưng cần tính latency, connector limits và nguồn chi phí.

Nguồn: [Athena Federated Query](https://docs.aws.amazon.com/athena/latest/ug/federated-queries.html).

#### NÊN làm (DO)

- **Dùng Athena cho ad hoc SQL/analytics trên log hoặc data lake ở S3** → Không cần quản lý query server; so với dựng cluster riêng cho truy vấn không thường xuyên, giảm vận hành.
- **Lưu dữ liệu columnar/compressed và partition theo access pattern** → Query đọc ít byte hơn; so với scan toàn bộ CSV/JSON mỗi lần, giảm độ trễ và scan cost.
- **Dùng Federated Query khi cần truy vấn nguồn ngoài S3 mà không muốn copy trước** → Connector gọi nguồn ngoài theo nhu cầu; so với ETL trước cho câu hỏi ad hoc, giảm bước chuẩn bị nhưng chấp nhận latency/connector constraints.

#### KHÔNG NÊN làm (DON'T)

- **Chạy `SELECT *` thường xuyên trên dữ liệu text lớn không partition** → Athena có thể scan nhiều data và tốn thời gian/chi phí; NÊN chọn cột, định dạng và partition phù hợp.
- **Coi Federated Query là miễn phí hoặc không có network dependency** → Connector Lambda, source database, VPC và query bytes đều có cost/limit; NÊN test workload và đo chi phí trước khi dùng production.

---

### 11. AWS Certificate Manager (ACM)

#### Giải thích cốt lõi

ACM cấp/quản lý TLS certificate và tích hợp với endpoint được hỗ trợ như Elastic Load Balancer, CloudFront và API Gateway. Managed renewal cần certificate/identity đáp ứng điều kiện validation và integration; certificate hết hạn khi validation hoặc resource association không còn đúng. Public certificate miễn phí theo slide nhưng private certificate/CA và dịch vụ tiêu thụ có thể có chi phí riêng. Nguồn: [ACM managed renewal](https://docs.aws.amazon.com/acm/latest/userguide/managed-renewal.html).

#### NÊN làm (DO)

- **Dùng ACM certificate cho AWS endpoint được tích hợp và theo dõi renewal status** → Service có thể dùng certificate lifecycle được quản lý; so với tải certificate thủ công, giảm thao tác rotate nhưng vẫn cần validation.
- **Duy trì DNS/email validation và alert trước certificate expiry** → ACM cần chứng minh domain control theo cơ chế validation; so với giả định “auto-renew nghĩa là không cần theo dõi”, phát hiện association/validation bị mất.
- **Đặt certificate ở đúng Region/service scope** → Một số integration yêu cầu certificate trong Region cụ thể; so với tái dùng ARN bất kỳ, tránh lỗi deployment do sai Region.

#### KHÔNG NÊN làm (DON'T)

- **Coi public certificate miễn phí nghĩa là toàn bộ private PKI cũng miễn phí** → ACM Private CA và thành phần khác có pricing riêng; NÊN phân biệt public certificate, private certificate và CA.
- **Đợi ngày hết hạn mới kiểm tra validation/renewal** → Certificate có thể không gia hạn nếu điều kiện validation/resource link hỏng; NÊN monitor renewal state và alarm sớm.

---

### 12. Amazon Macie

#### Giải thích cốt lõi

Macie dùng Machine Learning và pattern matching để khám phá dữ liệu nhạy cảm, đặc biệt trong S3, rồi tạo finding để đội ngũ điều tra. Nó hỗ trợ discovery/classification và cảnh báo, không tự thay thế IAM, encryption, retention hoặc quy trình xử lý incident. So với chỉ tìm tên bucket/object, phân tích nội dung có thể phát hiện PII nằm trong file; kết quả cần được xác minh theo ngữ cảnh vì pattern có thể có false positive/negative.

#### NÊN làm (DO)

- **Dùng Macie để tìm và ưu tiên review S3 data có thể chứa PII** → Tập trung điều tra vào vị trí có dấu hiệu nhạy cảm; so với rà từng object thủ công, tăng khả năng phát hiện trên quy mô lớn.
- **Kết hợp finding với owner, classification, access review và remediation workflow** → Phát hiện chỉ có ích khi có người xử lý; so với chỉ bật scan rồi để finding tồn đọng, giảm thời gian dữ liệu nhạy cảm bị phơi bày.
- **Đánh giá sampling/scope theo loại data và yêu cầu compliance** → Workload scan cần ưu tiên theo rủi ro; so với mặc định mọi bucket đều an toàn, đưa dữ liệu quan trọng vào phạm vi kiểm tra.

#### KHÔNG NÊN làm (DON'T)

- **Xem Macie là cơ chế ngăn mọi truy cập trái phép hoặc là bằng chứng chắc chắn không có PII** → Macie tạo finding, không thay authorization/prevention và detection không phải tuyệt đối; NÊN giữ IAM, encryption, logging và review dữ liệu.
- **Đóng finding mà không xác nhận object, owner và mức nhạy cảm** → False positive hoặc finding thật có thể bị xử lý sai; NÊN kiểm tra mẫu và ghi lại quyết định/biện pháp khắc phục.
