# Giải thích bổ sung - Section 15: AWS CI/CD

> *Phần này bổ sung kinh nghiệm thiết kế pipeline và vận hành; nội dung mở rộng được tách khỏi bản dịch slide.*

---

### 1. CI/CD: luồng thay đổi, môi trường và artifact

#### Giải thích cốt lõi

CI tạo feedback nhanh sau các lần push: lấy source, build, chạy kiểm thử và báo trạng thái. Continuous Delivery giữ phần mềm ở trạng thái có thể phát hành; approval có thể là bước thủ công. Continuous Deployment thường hàm ý tự động đưa thay đổi đạt điều kiện tới production. CodePipeline điều phối stage/action, còn CodeBuild, CodeDeploy hoặc dịch vụ đích thực thi từng phần việc. Artifact là đầu ra được chuyển giữa các stage; nếu stage sau build lại từ source thay vì dùng artifact đã kiểm tra, đầu vào deploy có thể khác với đầu vào đã test.

#### NÊN làm (DO)

- **Dùng cùng một artifact bất biến từ test đến deploy** → Mã được kiểm tra chính là mã được phát hành; so với build lại ở mỗi môi trường, giảm rủi ro khác dependency hoặc compiler output.
- **Tách source, build, test, approval và deploy thành action có đầu vào/đầu ra rõ** → Lỗi được khoanh vùng theo stage; so với một script lớn không có ranh giới, dễ retry và audit hơn.
- **Đặt approval trước production khi thay đổi có rủi ro hoặc cần kiểm soát tổ chức** → Có người chịu trách nhiệm xem kết quả test/diff; so với phát hành hoàn toàn tự động cho mọi thay đổi, giữ điểm kiểm soát cần thiết.
- **Tạo test phù hợp với từng lớp: unit, integration và smoke sau deploy** → Mỗi lớp phát hiện một loại lỗi khác nhau; so với chỉ coi build thành công là đủ, có bằng chứng cho behavior của phiên bản đã triển khai.
- **Theo dõi quyền của service role và nguồn artifact giữa các account** → Pipeline hoạt động bằng IAM principal riêng; so với gán quyền administrator, least privilege thu hẹp tác động khi cấu hình pipeline sai.

#### KHÔNG NÊN làm (DON'T)

- **Nhầm CI, Continuous Delivery và tự động deploy production là một việc** → Approval và điểm phát hành có thể khác; NÊN mô tả rõ pipeline nào chỉ tạo bản sẵn sàng phát hành, pipeline nào tự động triển khai.
- **Cho pipeline dùng quyền rộng hoặc artifact bucket chia sẻ không kiểm soát** → Người sửa pipeline/artifact có thể tác động production; NÊN tách role, bucket/prefix và quyền theo stage/account.
- **Để deploy thất bại mà không phát event/cảnh báo có chủ sở hữu** → Lỗi có thể nằm lâu và bản sửa không tới môi trường; NÊN dùng EventBridge/CloudWatch để thông báo và ghi rõ người xử lý.

---

### 2. Chọn repository và bảo vệ CodeCommit

#### Giải thích cốt lõi

CodeCommit là Git repository được AWS quản lý, có IAM, CloudTrail và tích hợp sâu với CodeBuild/CodePipeline. GitHub và các provider khác có hệ sinh thái, UI và tính năng cộng tác riêng; chọn nơi lưu source theo quy trình review, danh tính developer, yêu cầu compliance và khả năng tích hợp, không chỉ theo việc hạ tầng đã ở AWS.

Tình trạng mở cho khách hàng mới đã thay đổi theo thời gian: tài liệu CodeCommit ghi nhận đóng đăng ký khách hàng mới ngày 25-07-2024 và mở lại ngày 25-11-2025. Trước khi dùng, kiểm tra trạng thái/Region hiện tại và chính sách tổ chức. Nguồn chính thức: [CodeCommit User Guide history](https://docs.aws.amazon.com/codecommit/latest/userguide/history.html) và [The Future of AWS CodeCommit](https://aws.amazon.com/blogs/devops/aws-codecommit-returns-to-general-availability/).

#### NÊN làm (DO)

- **Dùng IAM Role/STS cho truy cập cross-account thay vì chia sẻ SSH key hoặc credentials** → Principal và session được quản lý theo account; so với credential dùng chung, dễ thu hồi và audit hơn.
- **Bật MFA/least privilege cho danh tính có quyền push, merge hoặc quản trị repository** → Thay đổi source có thể đi tới production; so với quyền rộng cho mọi developer, giảm nguy cơ xâm phạm pipeline.
- **Kiểm tra quota, file lớn và chính sách backup/retention dù slide mô tả repository không giới hạn kích thước** → “Không giới hạn” không nên hiểu là không có quota/kích thước file/giới hạn vận hành; so với chờ push/build thất bại, xác minh giới hạn hiện hành trước khi lưu binary lớn.
- **Đối chiếu CodeCommit với GitHub/GitLab theo nhu cầu review, UI, tích hợp và compliance** → Nhiều provider có thể kết nối CodePipeline; so với chọn vì quen thuộc hoặc vì AWS-only, quyết định theo tổng yêu cầu vận hành.
- **Kiểm tra tình trạng dịch vụ và Region trước khi đưa repository vào kiến trúc mới** → Availability/đăng ký dịch vụ có thể đổi; so với giả định từ slide cũ, kiểm tra tài liệu AWS hiện hành giảm rủi ro chọn nhầm dịch vụ.

#### KHÔNG NÊN làm (DON'T)

- **Chia sẻ access key hoặc SSH private key giữa developer/account** → Khó xác định ai thực hiện thao tác và credential khó thu hồi có chọn lọc; NÊN dùng IAM Role, STS và credential cá nhân có giới hạn.
- **Đưa secret vào Git history rồi chỉ xóa ở commit mới** → Lịch sử Git vẫn có thể chứa secret; NÊN revoke/rotate ngay, sau đó làm sạch lịch sử nếu cần và thêm secret scanning.
- **Xem Git hosting trong AWS đồng nghĩa source đã được bảo vệ tuyệt đối** → Quyền IAM, branch rule, token và pipeline vẫn có thể cấu hình sai; NÊN review quyền repository, approval rule, audit trail và pipeline role.

---

### 3. CodePipeline: stages, artifacts và role

#### Giải thích cốt lõi

CodePipeline điều phối source/build/test/deploy bằng stage và action. Một stage có thể chạy action tuần tự hoặc song song; manual approval tạo ranh giới phát hành. Artifact được đưa vào S3 và chuyển đến action kế tiếp, vì vậy bucket, encryption key và role của từng action đều là một phần của chuỗi tin cậy. EventBridge phù hợp để phản ứng khi pipeline/action/stage đổi trạng thái; CloudTrail trả lời ai đã gọi AWS API.

#### NÊN làm (DO)

- **Dùng manual approval ở ranh giới production hoặc thay đổi cần kiểm soát** → Người phê duyệt có thể xác nhận kết quả test và change window; so với cho mọi commit tự deploy, giảm lỗi phát hành không được xem xét.
- **Mã hóa artifact bucket bằng KMS và giới hạn quyền đọc/ghi theo pipeline** → Source/build output có thể chứa mã hoặc dữ liệu nhạy cảm; so với bucket công khai/dùng chung, giới hạn ai có thể lấy hay thay artifact.
- **Dùng artifact của stage đã test làm input của stage deploy** → Bảo đảm artifact đã được xác minh không bị thay bằng một bản khác; so với chạy lại build ở production, tăng tính lặp lại.
- **Tạo EventBridge rule cho failed/cancelled execution và gắn notification có người nhận** → Sự cố được phát hiện nhanh; so với chỉ trông chờ người mở Console, giảm thời gian không ai xử lý.
- **Kiểm tra IAM Service Role khi action không chạy được** → CodePipeline cần quyền gọi provider và truyền artifact; so với cấp `*` để “sửa nhanh”, thêm đúng action/resource cần thiết.

#### KHÔNG NÊN làm (DON'T)

- **Cho một stage ghi đè artifact sau khi stage khác đã xác nhận artifact** → Nội dung deploy có thể khác nội dung test; NÊN dùng versioned/immutable artifact và kiểm tra checksum/provenance khi quy trình yêu cầu.
- **Nhầm CloudTrail với log output của build** → CloudTrail ghi AWS API activity, không thay thế build log; NÊN dùng CloudWatch Logs/S3 cho output và CloudTrail cho audit API.
- **Bỏ qua trạng thái action song song khi một action lỗi** → Stage có thể dừng dù action khác đã hoàn thành; NÊN xác định cách retry/điều tra artifact của từng action trước khi chạy lại.

---

### 4. CodeBuild và `buildspec.yml`

#### Giải thích cốt lõi

CodeBuild cấp môi trường build theo project, lấy source, chạy lệnh trong container và ghi artifact/log. `buildspec.yml` mô tả environment, phases, artifacts và cache. Cache có thể giảm thời gian tải dependency nhưng không nên trở thành nguồn build chính; build phải tái lập từ lockfile và source. `parameter-store`/`secrets-manager` cho phép truy xuất secret theo IAM thay vì commit secret vào repository.

CodeBuild có compute/environment limits, IAM role và networking configuration. Dù slide mô tả mở rộng liên tục và không có build queue, concurrency/quota/account configuration vẫn có thể ảnh hưởng thời gian chờ. Dự án đặt trong VPC còn cần đường mạng tới dependency, registry và AWS API.

#### NÊN làm (DO)

- **Đặt `buildspec.yml` ở root hoặc cấu hình rõ đường dẫn trong project** → CodeBuild phải tìm đúng chỉ dẫn build; so với giả định file ở một thư mục khác, tránh build không chạy lệnh mong muốn.
- **Pin dependency và dùng cache chỉ như tối ưu** → Build vẫn có thể tái tạo từ lockfile khi cache mất; so với phụ thuộc vào cache để chứa package cần thiết, tránh kết quả không xác định.
- **Lấy secret từ Parameter Store/Secrets Manager và giới hạn role CodeBuild** → Secret không nằm trong Git hoặc buildspec; so với biến plaintext, giảm nguy cơ lộ qua source/log.
- **Tách phase để log lỗi rõ và chỉ export artifact cần thiết** → Dễ xác định lệnh install/test/build nào lỗi; so với script dài trong một command, dễ bảo trì và review hơn.
- **Kiểm tra VPC route, DNS, endpoint/NAT và quyền khi project không truy cập dependency** → CodeBuild chạy trong network đã cấu hình; so với mở toàn bộ egress, khắc phục đúng dependency và thu hẹp network access.
- **Theo dõi build duration, failure rate, concurrency và chi phí** → Compute tính phí theo thời gian và loại instance; so với tối ưu chỉ dựa trên một lần chạy, chọn environment theo workload thực tế.

#### KHÔNG NÊN làm (DON'T)

- **Đặt password/token production dưới `env.variables` plaintext** → Giá trị có thể lọt vào source/config/log; NÊN dùng `parameter-store` hoặc `secrets-manager` với quyền tối thiểu.
- **Upload toàn bộ workspace làm artifact mà không cần thiết** → Artifact phình to, có thể chứa credential/cache/temp file; NÊN whitelist output cần deploy và kiểm tra nội dung trước khi upload.
- **Tin rằng Docker container tự làm build an toàn hoặc hoàn toàn tái lập** → Image, package registry và dependency vẫn có thể bị thay đổi; NÊN pin image/dependency, cập nhật image có kiểm soát và scan artifact.
- **Cho CodeBuild role quyền administrator để mọi lệnh build đều chạy được** → Mã trong repository có thể sử dụng quyền đó; NÊN giới hạn role theo tài nguyên, hành động và stage.

---

### 5. CodeDeploy: deployment strategy và rollback

#### Giải thích cốt lõi

CodeDeploy dùng `appspec.yml` cùng deployment configuration để điều khiển lifecycle hook, target và tốc độ cập nhật. In-place cập nhật fleet hiện có; blue/green tạo fleet mới rồi chuyển traffic, thường cần Load Balancer cho EC2/ASG. Lambda traffic shifting chuyển alias theo canary/linear/all-at-once; ECS CodeDeploy dùng blue/green theo hợp đồng slide. Rollback của CodeDeploy là một deployment mới từ revision tốt gần nhất, không phải khôi phục nguyên trạng thái vật lý của deployment cũ.

#### NÊN làm (DO)

- **Chọn `OneAtATime`/tỷ lệ nhỏ khi capacity và downtime là ưu tiên; chọn `AllAtOnce` khi chấp nhận rủi ro để deploy nhanh** → Tốc độ triển khai và capacity còn phục vụ là trade-off; so với mặc định cùng một chiến lược cho mọi fleet, cấu hình theo SLO.
- **Dùng blue/green khi cần kiểm tra fleet mới trước khi chuyển traffic** → Fleet cũ còn khả năng nhận lại traffic trong thời gian kiểm tra; so với in-place, có đường chuyển đổi rõ hơn nhưng tốn thêm tài nguyên.
- **Đảm bảo CodeDeploy Agent và quyền đọc deployment bundle trên EC2/on-premises** → Agent thực hiện lifecycle trên host và cần lấy bundle; so với debug deployment failure sau khi rollout bắt đầu, preflight kiểm tra agent, network và S3 access.
- **Dùng lifecycle hooks để kiểm tra health sau các phase** → Có thể dừng/rollback khi bản mới không đạt tiêu chí; so với chỉ kiểm tra “deployment completed”, xác minh được behavior ứng dụng.
- **Dùng canary/linear và CloudWatch Alarm cho Lambda/ECS release quan trọng** → Tăng traffic theo giai đoạn và rollback khi metrics xấu; so với chuyển 100% ngay, giảm blast radius.
- **Kiểm tra kết quả rollback như một deployment riêng** → Revision cũ được redeploy; so với giả định có snapshot phục hồi tự động, xác minh lại trạng thái và health sau rollback.

#### KHÔNG NÊN làm (DON'T)

- **Chọn `HalfAtATime` mà không tính khả năng chịu tải khi mất nửa fleet** → Capacity có thể không đáp ứng traffic; NÊN tính headroom hoặc dùng tỷ lệ nhỏ hơn/blue-green.
- **Xem `appspec.yml` đúng là đủ nếu Agent, IAM hoặc bundle sai** → Deployment vẫn có thể thất bại ở host; NÊN kiểm tra đầy đủ file, agent, quyền và đường lấy artifact.
- **Coi rollback là revert dữ liệu hoặc side effect của ứng dụng** → CodeDeploy chỉ redeploy application revision; NÊN thiết kế migration dữ liệu tương thích ngược và kế hoạch recovery riêng.
- **Bật auto rollback nhưng không có alarm/health signal phù hợp** → Rollback không được kích hoạt đúng lúc hoặc bị kích hoạt sai; NÊN định nghĩa alarm theo lỗi/latency có ý nghĩa và kiểm thử trong staging.

---

### 6. CodeArtifact và quản trị dependency

#### Giải thích cốt lõi

CodeArtifact lưu package repository cho package manager như npm, pip, Maven, Gradle, NuGet, twine và yarn. Repository có thể dùng upstream/proxy tới public registry; CodeBuild và developer lấy dependency qua cùng hệ thống, giúp kiểm soát nguồn package. Domain gom repository; resource policy cho phép chia sẻ với account khác. EventBridge có thể phản ứng khi package version thay đổi để chạy kiểm tra hoặc pipeline.

#### NÊN làm (DO)

- **Dùng lockfile và quản lý package nguồn/phiên bản rõ ràng** → Build lặp lại cùng dependency; so với lấy package `latest` ở mỗi lần build, giảm rủi ro thay đổi ngoài ý muốn.
- **Dùng CodeArtifact làm nguồn package được kiểm soát và cache/proxy upstream theo chính sách** → Có thể tập trung kiểm soát dependency; so với developer tải trực tiếp từ nhiều registry, dễ audit nguồn hơn.
- **Giới hạn resource policy cross-account theo role/account và quyền đọc cần thiết** → Package có thể là tài sản nội bộ; so với chia sẻ repository rộng, giảm phạm vi lộ package.
- **Dùng EventBridge để chạy scan/build/test khi package mới hoặc thay đổi** → Có thể phát hiện dependency mới có lỗi/security issue; so với chờ release ứng dụng mới, phát hiện sớm hơn.

#### KHÔNG NÊN làm (DON'T)

- **Tin rằng repository riêng tư tự bảo đảm package an toàn** → Package nội bộ có thể chứa dependency dễ tổn thương hoặc bị nhầm nguồn; NÊN scan, pin version và review người publish.
- **Cho pipeline deploy thẳng package mới chỉ vì có event** → Tạo/sửa package không chứng minh package đã qua test; NÊN đặt validation stage trước release.
- **Chia sẻ domain/repository cross-account bằng policy không review** → Principal ngoài account có thể truy cập package; NÊN xác định account, repository, read/write và audit policy.

---

### 7. CodeGuru Reviewer và Profiler

#### Giải thích cốt lõi

CodeGuru Reviewer phân tích mã tĩnh để gợi ý bug, security issue hoặc best-practice issue trong quá trình phát triển. CodeGuru Profiler thu thập profile khi ứng dụng chạy để tìm CPU/memory hotspot, heap usage và anomaly. Reviewer trả lời “đoạn mã nào có rủi ro?”, Profiler trả lời “ứng dụng đang tiêu tốn tài nguyên ở đâu?”. Cả hai là nguồn hỗ trợ review/điều tra, không thay thế test, threat model hoặc profiling có mục tiêu.

#### NÊN làm (DO)

- **Dùng Reviewer để bổ sung review cho repository/ngôn ngữ được hỗ trợ** → Có thể phát hiện vấn đề khó thấy qua review thủ công; so với chỉ chạy linter, thêm phân tích chuyên biệt nhưng vẫn cần người đánh giá finding.
- **Dùng Profiler khi cần xác định CPU/memory hotspot trong workload thực** → Profile dựa trên runtime behavior; so với tối ưu hóa theo phỏng đoán, tập trung vào code path tiêu tốn tài nguyên.
- **Điều chỉnh sampling/reporting theo yêu cầu độ chi tiết, overhead và thời gian quan sát** → Sampling cao cho nhiều dữ liệu hơn nhưng có thể tăng overhead; so với giữ một cấu hình mặc định cho mọi workload, cân bằng theo mục tiêu vận hành.
- **Đánh giá finding trong ngữ cảnh ứng dụng rồi xác minh bằng test/benchmark** → Khuyến nghị tự động có thể cần phân loại; so với áp dụng máy móc mọi gợi ý, giảm nguy cơ thay đổi không cần thiết.

#### KHÔNG NÊN làm (DON'T)

- **Coi CodeGuru Reviewer là chứng nhận mã không có lỗ hổng** → Static analysis không quan sát mọi runtime path hoặc yêu cầu nghiệp vụ; NÊN kết hợp test, threat modeling và security review.
- **Tối ưu chỉ theo một lần profile hoặc chỉ theo heap summary** → Workload và traffic thay đổi theo thời gian; NÊN profile đại diện production workload và xác minh cải thiện bằng benchmark.
- **Tăng sampling/reporting mà không theo dõi overhead và chi phí** → Dữ liệu profile nhiều hơn có thể tăng tài nguyên/telemetry; NÊN điều chỉnh `SamplingIntervalInMilliseconds`, reporting interval và giới hạn memory có kiểm soát.
