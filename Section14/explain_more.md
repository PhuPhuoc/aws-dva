# Giải thích bổ sung - Section 14: AWS Serverless, SAM, CDK, Cognito và các dịch vụ liên quan

> *Phần này bổ sung kiến thức triển khai thực tế; nội dung mở rộng được tách khỏi bản dịch slide.*

---

### 1. SAM: template, triển khai và đồng bộ mã

#### Giải thích cốt lõi

SAM là lớp khai báo serverless được CloudFormation transform thành template triển khai. Resource như `AWS::Serverless::Function` rút gọn một số cấu hình Lambda, nhưng stack vẫn được quản lý bởi CloudFormation khi triển khai theo luồng thông thường. `sam build`, `sam package` và `sam deploy` phục vụ các bước chuẩn bị artifact và cập nhật stack; `sam local` mô phỏng một phần hành vi Lambda/API trên máy phát triển, không thay thế kiểm thử tích hợp trong AWS.

`sam sync --code` tối ưu vòng lặp phát triển bằng cách cập nhật mã qua API dịch vụ mà không cập nhật stack CloudFormation. Tốc độ nhanh hơn nhưng trạng thái mã trên AWS có thể tạm thời khác với artifact/template được quản lý bởi pipeline. Policy templates rút ngắn việc khai báo quyền, song quyền thực tế vẫn cần được kiểm tra theo tài nguyên và hành động ứng dụng dùng.

#### NÊN làm (DO)

- **Dùng `sam deploy` hoặc pipeline CloudFormation làm luồng phát hành chính** → Stack, cấu hình và mã được triển khai theo một thay đổi có thể truy vết; so với cập nhật thủ công, dễ review và khôi phục hơn.
- **Dùng `sam sync --code` cho vòng lặp phát triển nhanh ở môi trường dev** → Bỏ qua bước cập nhật hạ tầng nên phản hồi nhanh; so với dùng lệnh này làm nguồn triển khai production, giữ cho trạng thái CloudFormation và mã triển khai có thể được đồng bộ lại.
- **Kiểm tra quyền được sinh từ SAM policy template theo least privilege** → Template chỉ là lối tắt khai báo, không hiểu đầy đủ ranh giới nghiệp vụ; so với giữ quyền rộng mặc định, giới hạn action/resource giảm tác động khi function bị khai thác.
- **Kết hợp SAM Local với kiểm thử integration trên AWS** → Local giúp kiểm tra handler, route và event nhanh; so với xem kết quả local là bằng chứng đầy đủ, integration test phát hiện khác biệt về IAM, dịch vụ thật và cấu hình môi trường.
- **Dùng CodeDeploy alias, canary và alarm cho Lambda quan trọng** → Traffic được tăng dần và có tín hiệu để rollback; so với chuyển toàn bộ traffic ngay, giảm phạm vi ảnh hưởng của bản lỗi.

#### KHÔNG NÊN làm (DON'T)

- **Coi `sam sync --code` là thay thế triển khai hạ tầng trong production** → Hạ tầng trong CloudFormation có thể không khớp trạng thái đang chạy; NÊN dùng lệnh này cho phát triển nhanh và phát hành chính thức qua deployment có quản lý stack.
- **Giả định chạy được với SAM Local đồng nghĩa hoạt động giống hệt AWS** → Mô phỏng không bao quát mọi quyền, giới hạn và tích hợp dịch vụ; NÊN bổ sung kiểm thử trong tài khoản/môi trường test.
- **Dùng policy template mà không rà soát quyền đã cấp** → Function có thể nhận quyền vượt nhu cầu; NÊN giới hạn resource/action và kiểm tra template đã transform.
- **Đưa secret vào SAM template hoặc biến môi trường dạng plaintext** → Secret có thể lọt vào source control, artifact hoặc log; NÊN lấy secret từ Secrets Manager/Parameter Store và giới hạn IAM access.

---

### 2. CDK: construct, CloudFormation và vòng đời triển khai

#### Giải thích cốt lõi

CDK cho phép mô tả hạ tầng bằng mã TypeScript/JavaScript, Python, Java hoặc .NET. Construct đóng gói resource ở các mức trừu tượng khác nhau; `cdk synth` tổng hợp mã thành CloudFormation template. CloudFormation vẫn là lớp thực hiện thay đổi tài nguyên, nên template tổng hợp, IAM permission, replacement và deletion vẫn cần review như hạ tầng khai báo.

`cdk bootstrap` tạo tài nguyên dùng cho triển khai vào từng AWS environment (account và Region), thường gồm S3 bucket và IAM Role trong stack `CDKToolkit`. CDK Assertions kiểm thử template tổng hợp; fine-grained assertions nhắm vào thuộc tính cụ thể, còn snapshot kiểm tra thay đổi toàn template và dễ tạo nhiễu khi nhiều thuộc tính vô can thay đổi.

#### NÊN làm (DO)

- **Chạy `cdk synth` và review template trước khi triển khai** → Đây là đầu ra CloudFormation sẽ được dùng để tạo/cập nhật stack; so với chỉ đọc mã construct, review template giúp thấy resource, policy và giá trị cuối cùng.
- **Chạy `cdk diff` trước thay đổi môi trường dùng chung** → Hiển thị khác biệt giữa mã hiện tại và stack; so với triển khai ngay, có cơ hội phát hiện resource replacement hoặc thay đổi quyền ngoài dự kiến.
- **Bootstrap mỗi account/Region đích trước lần triển khai đầu tiên** → Toolkit cần bucket và role trong environment; so với chờ deploy thất bại, bootstrap chủ động làm rõ quyền và tài nguyên staging.
- **Viết fine-grained assertions cho các thuộc tính bảo mật/hành vi quan trọng** → Kiểm thử tập trung vào điều kiện cần giữ; so với chỉ snapshot toàn template, ít bị nhiễu bởi thay đổi không liên quan.
- **Dùng CDK khi cần ngôn ngữ lập trình và abstraction cho nhiều loại tài nguyên; dùng SAM khi trọng tâm là serverless** → Chọn theo độ phức tạp và khả năng bảo trì; so với chọn công cụ theo sở thích ngôn ngữ, tránh thêm abstraction không cần thiết.

#### KHÔNG NÊN làm (DON'T)

- **Coi construct cấp cao là bằng chứng resource đã có cấu hình an toàn** → Abstraction có thể tạo nhiều resource hoặc quyền gián tiếp; NÊN kiểm tra template synth, IAM policy và `cdk diff`.
- **Chạy `cdk deploy` mà không xem xét diff ở production** → Thay đổi có thể thay thế hoặc xóa tài nguyên; NÊN review diff và yêu cầu approval cho thay đổi có tác động dữ liệu.
- **Dùng snapshot tests làm kiểm thử duy nhất cho mọi yêu cầu** → Snapshot chỉ phát hiện sai khác tổng thể, không nhất thiết diễn đạt rõ invariant; NÊN thêm assertion cụ thể cho policy, encryption, network và resource quan trọng.
- **Xem `cdk destroy` là thao tác dọn dẹp vô hại** → Lệnh có thể xóa tài nguyên và dữ liệu của stack; NÊN xác nhận account, Region, stack và chính sách bảo vệ dữ liệu trước khi chạy.
- **Sao chép nguyên văn lệnh cài đặt CLI trong slide mà không kiểm tra package và phiên bản CDK** → CLI và thư viện ứng dụng là hai package khác nhau; NÊN xác minh package/lệnh cài đặt trong tài liệu CDK tương ứng với dự án trước khi thiết lập môi trường.

---

### 3. Cognito User Pools và Identity Pools

#### Giải thích cốt lõi

Cognito User Pool quản lý người dùng ứng dụng và quy trình đăng nhập; sau khi xác thực, client nhận token (thường là JWT). Identity Pool nhận danh tính đã liên kết/federated và cấp temporary AWS credentials thông qua IAM Role để client có thể gọi dịch vụ AWS trực tiếp. Vì vậy, cách nhớ trong đề thi là **User Pool xác minh người dùng ứng dụng; Identity Pool đổi danh tính thành AWS credentials**.

Hai khái niệm này không tự động giải quyết toàn bộ authorization nghiệp vụ. Token hợp lệ cho biết danh tính/claims, còn API/backend vẫn phải quyết định principal đó được phép làm gì trên tài nguyên cụ thể. Identity Pool role và policy cần giới hạn chặt, đặc biệt với guest access. Lambda trigger chạy đồng bộ có thể nằm ngay trên đường đăng nhập/đăng ký nên ảnh hưởng thời gian đáp ứng và khả năng hoàn tất luồng.

#### NÊN làm (DO)

- **Dùng User Pool cho đăng ký, đăng nhập và quản lý người dùng ứng dụng** → Dịch vụ quản lý credential, MFA, liên kết danh tính và token; so với tự xây toàn bộ hệ thống đăng nhập, giảm mã xác thực phải tự vận hành.
- **Dùng Identity Pool khi client cần temporary AWS credentials để gọi AWS service** → Cognito ánh xạ danh tính sang IAM Role; so với nhúng access key dài hạn trong mobile/web app, credentials có thời hạn và quyền có thể giới hạn.
- **Dùng cả User Pool và Identity Pool khi cần vừa đăng nhập ứng dụng vừa truy cập AWS trực tiếp** → User Pool xác thực, Identity Pool cấp credentials; so với nhầm lẫn hai pool, phân chia rõ token ứng dụng và quyền AWS.
- **Kiểm tra quyền trên từng hành động/tài nguyên ở backend sau khi xác minh token** → Đăng nhập thành công không đồng nghĩa được phép đọc mọi bản ghi; so với chỉ kiểm tra token, kiểm tra nghiệp vụ chặn truy cập ngang giữa người dùng.
- **Giữ Lambda trigger ngắn, có thể quan sát và an toàn khi lỗi** → Trigger đồng bộ có thể trì hoãn hoặc chặn đăng nhập/đăng ký; so với xử lý nặng trong trigger, chuyển việc không thiết yếu sang xử lý bất đồng bộ sau sự kiện.
- **Giới hạn guest role và kiểm tra claims/policy variable trước khi cấp quyền** → Guest cũng có thể nhận credentials nếu bật anonymous access; so với policy rộng, least privilege giảm khả năng lạm dụng.

#### KHÔNG NÊN làm (DON'T)

- **Dùng User Pool với kỳ vọng nó tự cấp AWS credentials cho client** → User Pool chủ yếu trả token danh tính; NÊN thêm Identity Pool nếu ứng dụng cần temporary AWS credentials.
- **Dùng Identity Pool như một hệ thống thay thế việc quản lý đăng nhập ứng dụng** → Identity Pool tập trung liên kết danh tính và cấp role; NÊN dùng User Pool hoặc IdP phù hợp cho quy trình sign-up/sign-in.
- **Coi JWT hợp lệ là quyền truy cập hợp lệ vào mọi tài nguyên** → Token chứng minh danh tính/claims chứ không thay thế kiểm tra quyền nghiệp vụ; NÊN xác thực authorization tại API/backend.
- **Cấp quyền rộng cho authenticated hoặc unauthenticated role** → Client có thể gọi thêm dịch vụ/tài nguyên không cần thiết; NÊN cấp action và resource nhỏ nhất, kiểm thử cả luồng guest.
- **Đặt thao tác chậm hoặc phụ thuộc không ổn định vào Lambda trigger đồng bộ** → Login có thể timeout hoặc thất bại theo dependency; NÊN giữ trigger tối giản và chuyển công việc phụ sang xử lý bất đồng bộ khi luồng cho phép.

---

### 4. Step Functions: orchestration, lỗi và callback

#### Giải thích cốt lõi

Step Functions điều phối workflow qua state machine. Task State gọi một công việc/service; Choice, Wait, Map và Parallel điều khiển luồng; Retry/Catch diễn tả chính sách phục hồi gần với workflow thay vì nhồi toàn bộ điều phối vào mã Lambda. Standard và Express khác nhau về thời lượng, cách ghi execution history, thông lượng, giá và ngữ nghĩa giao nhận. Chọn theo yêu cầu về thời gian chạy, audit, tốc độ và cách xử lý duplicate; không chỉ theo số lượng request.

`Wait for Task Token` tạm dừng workflow để hệ thống ngoài hoặc người duyệt gọi lại với token. Activity Task dùng worker poll và heartbeat. Những mô hình này kéo dài thời gian chờ nhưng cần bảo vệ token, đặt timeout và xử lý callback đến trễ/lặp.

#### NÊN làm (DO)

- **Dùng Step Functions để điều phối các bước có trạng thái, nhánh, retry hoặc chờ dài** → Tiến trình và lịch sử được thể hiện trong state machine; so với một Lambda dài chứa mọi bước, dễ quan sát và phục hồi từng bước hơn.
- **Chọn Standard cho workflow dài hoặc cần lịch sử execution rõ; chọn Express cho workflow ngắn, thông lượng cao sau khi xác minh semantics** → Hai loại khác về thời lượng, history và giao nhận; so với chọn mặc định, khớp lựa chọn với yêu cầu audit và duplicate handling.
- **Thiết kế tác vụ downstream idempotent, kể cả khi dùng Standard** → Retry, timeout hoặc kết quả callback không rõ có thể khiến ứng dụng gọi lại thao tác; so với giả định workflow đảm bảo side effect đúng một lần, idempotency bảo vệ thanh toán và ghi dữ liệu.
- **Đặt Retry cụ thể cho lỗi tạm thời rồi Catch để chuyển lỗi không phục hồi được** → Backoff giảm áp lực lên dependency và Catch giữ đường xử lý lỗi; so với retry mọi exception, tránh retry lỗi cấu hình hoặc dữ liệu vĩnh viễn.
- **Dùng `ResultPath` để giữ input cần thiết khi chuyển sang nhánh lỗi** → Context gốc và error object có thể cùng được chuyển tiếp; so với ghi đè toàn bộ input, nhánh xử lý vẫn có dữ liệu để ghi log/compensate.
- **Bảo vệ Task Token và kiểm tra callback/heartbeat trong luồng dài** → Token là khả năng tiếp tục đúng execution; so với token công khai hoặc worker im lặng, giới hạn lộ token và phát hiện task treo sớm hơn.

#### KHÔNG NÊN làm (DON'T)

- **Diễn giải “exactly-once execution” thành đảm bảo side effect bên ngoài chỉ xảy ra một lần** → Timeout/response mất có thể làm client không biết thao tác đã thành công chưa; NÊN dùng idempotency key hoặc điều kiện ghi duy nhất ở hệ thống đích.
- **Đặt `States.ALL` trước các retry/catch cụ thể** → Rule rộng có thể bắt lỗi trước rule hẹp; NÊN sắp xếp rule cụ thể trước và rule tổng quát sau.
- **Retry lỗi vĩnh viễn như lỗi tạm thời** → Retry làm tăng chi phí và trì hoãn kết quả mà không sửa input/quyền sai; NÊN lọc `ErrorEquals`, đặt số lần và thời gian chờ theo dependency.
- **Nuốt lỗi trong Catch rồi đánh dấu workflow thành công** → Monitoring sẽ báo sai trạng thái và consumer không biết kết quả bị thiếu; NÊN chuyển lỗi sang state xử lý, ghi nhận trạng thái thất bại hoặc phát cảnh báo rõ ràng.
- **Đưa Task Token vào URL/log công khai hoặc không đặt timeout cho Activity** → Người ngoài có thể tiếp tục execution hoặc task có thể chờ vô hạn; NÊN truyền token qua kênh tin cậy, giới hạn quyền và đặt timeout/heartbeat.
- **Chọn Express async mà giả định có thể nhận kết quả ngay, hoặc quên xử lý giao nhận lặp** → Async không chờ workflow hoàn tất và at-least-once có thể tạo duplicate; NÊN dùng callback/result phù hợp và làm consumer idempotent.

---

### 5. AppSync và GraphQL API

#### Giải thích cốt lõi

AppSync là dịch vụ GraphQL managed. GraphQL schema định nghĩa hợp đồng; resolver ánh xạ operation đến data source như DynamoDB, Aurora, OpenSearch, HTTP API hoặc Lambda. Client yêu cầu trường cần thiết thay vì nhận một response REST cố định. Subscription hỗ trợ cập nhật realtime; ứng dụng mobile có thể dùng tính năng local/offline và đồng bộ.

AppSync có nhiều authorization mode: API key, IAM, OIDC và Cognito User Pools. Chọn mode theo loại client/danh tính; resolver và backend vẫn cần bảo vệ dữ liệu theo người dùng. GraphQL cho phép client chọn shape của response nhưng không tự động giới hạn chi phí truy vấn, độ sâu hay quyền trên từng record.

#### NÊN làm (DO)

- **Thiết kế schema theo hợp đồng dữ liệu mà client thực sự cần** → Client chỉ yêu cầu field cần thiết và có thể kết hợp nhiều nguồn; so với endpoint riêng cho mọi biến thể, giảm số lần round-trip và payload dư.
- **Chọn authorization mode theo danh tính: IAM cho workload AWS, Cognito/OIDC cho user token** → Cơ chế phù hợp với issuer và principal; so với dùng API key như credential cho mọi client, ranh giới xác thực rõ hơn.
- **Kiểm tra quyền tại resolver hoặc data source cho từng user/resource** → Có thể có nhiều loại client cùng truy vấn schema; so với chỉ ẩn nút trên UI, kiểm tra phía server ngăn gọi trực tiếp field không được phép.
- **Dùng batching/caching phù hợp khi resolver gọi nhiều nguồn** → Giảm fan-out và truy vấn lặp; so với resolver tạo một lời gọi cho mỗi record, tránh tăng latency và tải dependency.
- **Đặt giới hạn và quan sát query/subscription behavior** → Truy vấn lồng sâu hoặc subscription dài có thể tạo tải; so với mở schema không giới hạn, hạn chế độ phức tạp và phát hiện bất thường sớm.

#### KHÔNG NÊN làm (DON'T)

- **Coi GraphQL schema là cơ chế phân quyền** → Schema cho biết field tồn tại, không quyết định user nào được đọc record nào; NÊN kiểm tra claims và quyền trong resolver/backend.
- **Coi `API_KEY` là xác thực mạnh cho dữ liệu riêng tư** → API key có thể bị chia sẻ hoặc lộ và không đại diện một danh tính user; NÊN dùng IAM, Cognito User Pools hoặc OIDC cho dữ liệu cần bảo vệ.
- **Cho phép resolver trả toàn bộ record trong khi client chỉ cần một vài field** → Có thể lộ dữ liệu nhạy cảm và tăng payload; NÊN trả đúng field được phép theo schema/authorization.
- **Bật realtime/offline mà không xác định quy tắc xung đột và quyền đồng bộ** → Dữ liệu cũ hoặc quyền thay đổi có thể làm client giữ/cập nhật sai dữ liệu; NÊN xác định ownership, conflict handling và cách thu hồi quyền.

---

### 6. AWS Amplify cho ứng dụng frontend

#### Giải thích cốt lõi

Amplify là tập hợp công cụ và thư viện để xây dựng, kết nối, build và host ứng dụng web/mobile; các khả năng phía sau dựa trên dịch vụ AWS như Cognito, AppSync và DynamoDB. Amplify Hosting tập trung vào build/deploy frontend, preview và domain. DataStore cung cấp mô hình làm việc với dữ liệu cục bộ và đồng bộ với backend. Amplify giúp rút ngắn phần tích hợp, nhưng không loại bỏ trách nhiệm thiết kế schema, IAM, authorization và kiểm thử dịch vụ nền.

#### NÊN làm (DO)

- **Dùng Amplify khi ứng dụng cần luồng frontend tích hợp nhanh với dịch vụ AWS** → Có thư viện, component và hosting hỗ trợ sẵn; so với tự nối từng dịch vụ từ đầu, giảm mã boilerplate.
- **Xem rõ dịch vụ AWS mà từng tính năng Amplify tạo/dùng** → Authentication, DataStore và Hosting có dependency khác nhau; so với chỉ nhìn lớp wrapper, dễ kiểm soát chi phí, quyền và vận hành backend.
- **Chạy E2E test trong bước build trước khi deploy** → Có thể phát hiện regression ở luồng UI tích hợp; so với chỉ kiểm thử unit, xác minh được luồng người dùng trước khi phát hành.
- **Quản lý custom domain, redirect và header cùng cấu hình triển khai** → Môi trường preview và production có thể khác nhau; so với chỉnh thủ công sau deploy, cấu hình được review và lặp lại.

#### KHÔNG NÊN làm (DON'T)

- **Coi Amplify là dịch vụ thay thế mọi quyết định backend hoặc security** → Các dịch vụ AWS nền vẫn cần IAM, authorization và thiết kế dữ liệu đúng; NÊN review cấu hình được sinh ra và quyền của từng môi trường.
- **Giả định dữ liệu local/offline luôn đồng bộ tức thời và không xung đột** → Client có thể offline hoặc cập nhật đồng thời; NÊN xác định hành vi merge/conflict và xử lý trạng thái đồng bộ trong UI.
- **Trộn hướng dẫn CLI/config của các thế hệ Amplify mà không kiểm tra phiên bản dự án** → Cấu trúc cấu hình và lệnh có thể khác giữa các thế hệ; NÊN khóa phiên bản và theo tài liệu phù hợp với project hiện tại.
- **Đưa secret production vào frontend bundle hoặc biến public của build** → Nội dung frontend có thể tải xuống bởi bất kỳ client nào; NÊN chỉ đưa cấu hình public vào bundle và giữ secret ở backend/secret store.
