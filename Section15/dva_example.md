# Câu hỏi ôn tập DVA-C02 - Section 15: AWS CI/CD

> *Câu hỏi và lựa chọn bằng tiếng Anh; phần giải thích bằng tiếng Việt.*

---

## 1. Nhận diện Continuous Integration

### Question 1:
A team wants every code push to trigger an automated build and test, then provide developers with quick feedback about failures. Which practice describes this workflow?

```
A. Continuous Integration
B. Blue/green deployment
C. Package management
D. Runtime profiling
```

### ✅ Answer: **A. Continuous Integration**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- CI là thực hành developer thường xuyên push mã và build/test server kiểm tra thay đổi ngay, sau đó trả feedback.
- Mục tiêu là phát hiện lỗi sớm và giảm thời gian chờ feedback.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Blue/green là chiến lược deployment để chuyển traffic giữa hai môi trường, không mô tả build/test mỗi lần push. |
| C | Package management lưu/truy xuất dependency, không phải quy trình CI. |
| D | Runtime profiling đo hành vi ứng dụng khi chạy, không kiểm thử code sau mỗi push. |

### 📝 Vocabulary Analysis:
- **code push**: lần developer đẩy commit lên repository.
- **quick feedback**: phản hồi nhanh về build/test.
- **failure**: bước kiểm tra thất bại.

### 🔄 Context Variations:
- **If the software must remain ready to release when requested** → Đó là mục tiêu Continuous Delivery.
- **If every passing change is automatically released to production** → Đó là mức tự động hóa Continuous Deployment.

---

## 2. Chọn dịch vụ điều phối CI/CD

### Question 2:
A company needs a visual workflow that connects a source repository, build and test actions, a manual approval, and a deployment action. Which AWS service should orchestrate the stages?

```
A. AWS CodePipeline
B. AWS CodeGuru Profiler
C. AWS CodeArtifact
D. AWS Systems Manager Session Manager
```

### ✅ Answer: **A. AWS CodePipeline**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- CodePipeline điều phối workflow CI/CD qua các stage và action.
- Stage có thể có action tuần tự/song song và có thể đặt manual approval.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | CodeGuru Profiler phân tích hiệu năng runtime, không điều phối pipeline. |
| C | CodeArtifact quản lý software package/dependency, không chạy các stage deploy. |
| D | Session Manager mở phiên quản trị instance, không điều phối CI/CD. |

### 📝 Vocabulary Analysis:
- **orchestrate**: điều phối nhiều bước/dịch vụ theo một workflow.
- **manual approval**: bước cần người phê duyệt trước khi tiếp tục.
- **deployment action**: action triển khai artifact lên môi trường đích.

### 🔄 Context Variations:
- **If the requirement is to execute build commands in a managed environment** → Dùng CodeBuild trong pipeline.
- **If only the application deployment lifecycle needs automation** → CodeDeploy xử lý deployment, thường được gọi từ CodePipeline.

---

## 3. Truyền artifact giữa các stage

### Question 3:
A CodePipeline build action produces a deployment package. The next stage must consume the same package that was built. Where does CodePipeline store and pass stage artifacts according to the course material?

```
A. An S3 bucket
B. An IAM user access key
C. A Cognito User Pool
D. A CodeGuru profile
```

### ✅ Answer: **A. An S3 bucket**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Artifact được lưu trong S3 bucket và chuyển từ stage này sang stage tiếp theo.
- Dùng cùng artifact giúp stage deploy nhận đúng đầu ra của build action.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Access key là thông tin xác thực, không phải nơi lưu artifact. |
| C | User Pool quản lý danh tính người dùng ứng dụng. |
| D | CodeGuru profile là dữ liệu profiling, không phải deployment package. |

### 📝 Vocabulary Analysis:
- **deployment package**: artifact được triển khai lên môi trường đích.
- **consume the same package**: stage sau dùng đúng artifact do stage trước tạo.
- **stage artifact**: đầu ra được truyền giữa các stage.

### 🔄 Context Variations:
- **If the pipeline uses a third-party artifact source** → Cấu hình source action tương ứng, nhưng artifact stage vẫn được quản lý theo pipeline.
- **If the artifact contains sensitive code/configuration** → Mã hóa bucket và giới hạn quyền IAM/KMS.

---

## 4. Truy cập CodeCommit cross-account an toàn

### Question 4:
A developer in one AWS account needs temporary access to a CodeCommit repository in another account. What is the recommended approach in the course material?

```
A. Share the developer's SSH private key with the target account
B. Share long-term AWS access keys
C. Assume an IAM role in the target account using AWS STS
D. Make the repository public
```

### ✅ Answer: **C. Assume an IAM role in the target account using AWS STS**

### 📖 Explanation (Vietnamese):

**Tại sao chọn C:**
- Slide khuyến nghị không chia sẻ SSH key hoặc AWS credentials.
- Dùng IAM Role và STS `AssumeRole` để nhận temporary credentials có thể kiểm soát/audit.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| A | Chia sẻ private key làm mất khả năng quy trách nhiệm và khó thu hồi riêng cho một người. |
| B | Long-term key có thể tồn tại lâu sau khi không còn cần; không theo least-privilege session. |
| D | Public repository mở mã nguồn, không phải cách cấp cross-account access có kiểm soát. |

### 📝 Vocabulary Analysis:
- **temporary access**: quyền truy cập có thời hạn.
- **AssumeRole**: dùng STS để nhận session credentials của IAM Role được phép.
- **target account**: account đang chứa repository cần truy cập.

### 🔄 Context Variations:
- **If the developer is in the same account** → Cấp IAM policy phù hợp cho user/role trên repository.
- **If access must be revoked** → Thu hồi quyền trust/policy hoặc hết hạn session thay vì tìm và thay credential dùng chung.

---

## 5. Chọn giữa CodeCommit và GitHub

### Question 5:
A team requires a Git repository integrated with AWS IAM identities and hosted as a managed AWS service. Which option best matches the requirement in the course comparison?

```
A. AWS CodeCommit
B. GitHub Enterprise self-hosted only
C. AWS CodeArtifact
D. Amazon CodeGuru Reviewer
```

### ✅ Answer: **A. AWS CodeCommit**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Theo bảng trong slide, CodeCommit được AWS quản lý/host và sử dụng IAM user/role làm danh tính.
- CodeCommit và GitHub đều hỗ trợ pull request, CodeBuild integration và SSH/HTTPS; yếu tố phân biệt ở câu hỏi là IAM và hosting do AWS quản lý.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | GitHub Enterprise self-hosted là lựa chọn host trên server riêng, không phải Git repository được AWS quản lý. |
| C | CodeArtifact quản lý dependency/package, không phải Git source repository. |
| D | Reviewer phân tích code, không host repository. |

### 📝 Vocabulary Analysis:
- **managed service**: dịch vụ được provider vận hành phần hạ tầng.
- **IAM identity**: AWS user/role dùng để xác định principal và quyền.
- **self-hosted**: khách hàng tự host và vận hành trên server của mình.

### 🔄 Context Variations:
- **If the team prioritizes GitHub’s full-featured UI and ecosystem** → GitHub có thể phù hợp; slide mô tả UI của GitHub đầy đủ tính năng hơn.
- **If a new account plans to adopt CodeCommit** → Kiểm tra tình trạng đăng ký/Region hiện hành; AWS đã thay đổi chính sách mở cho khách hàng mới theo thời gian.

---

## 6. Chọn tệp hướng dẫn cho CodeBuild

### Question 6:
A CodeBuild project needs to install dependencies, run pre-build commands, compile the application, upload selected outputs, and cache dependencies for future builds. Which file should define these phases and outputs?

```
A. appspec.yml
B. buildspec.yml
C. package.json only
D. cdk.out
```

### ✅ Answer: **B. `buildspec.yml`**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- `buildspec.yml` định nghĩa `env`, các `phases`, `artifacts` và `cache` cho CodeBuild.
- Slide yêu cầu file này nằm ở root của source code, trừ khi project được cấu hình đường dẫn khác.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| A | `appspec.yml` mô tả deployment của CodeDeploy, không phải command build/test của CodeBuild. |
| C | `package.json` quản lý dependency/script của một số dự án Node.js, nhưng không thay thế buildspec cho phases/artifacts CodeBuild. |
| D | `cdk.out` chứa template tổng hợp của CDK, không mô tả lệnh CodeBuild. |

### 📝 Vocabulary Analysis:
- **pre-build**: bước chuẩn bị trước lệnh build chính.
- **selected outputs**: các file cần đưa vào artifact.
- **cache dependencies**: lưu dependency để rút ngắn lần build tiếp theo.

### 🔄 Context Variations:
- **If the commands are entered directly in the CodeBuild Console** → Slide cho phép cấu hình build instructions trong Console thay vì file.
- **If CodeDeploy needs lifecycle instructions** → Dùng `appspec.yml`.

---

## 7. Lấy secret cho CodeBuild

### Question 7:
A build needs a database password. The team wants to avoid committing the password to Git or storing it as plaintext in the buildspec. Which configuration source should the team use?

```
A. The `secrets-manager` or `parameter-store` environment mapping
B. A public variable in the repository
C. A line printed by the build command
D. The CodePipeline stage name
```

### ✅ Answer: **A. The `secrets-manager` or `parameter-store` environment mapping**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- `buildspec.yml` hỗ trợ `secrets-manager` và `parameter-store` để lấy giá trị từ kho secret/config thay vì commit plaintext.
- CodeBuild role cần quyền tối thiểu để đọc đúng secret/parameter.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Giá trị trong repository có thể bị đọc qua lịch sử Git và pull request. |
| C | In secret ra log có thể làm lộ credential cho người có quyền đọc log. |
| D | Stage name không phải secret store. |

### 📝 Vocabulary Analysis:
- **committing a password**: đưa password vào version control.
- **plaintext**: dữ liệu chưa được bảo vệ/mã hóa.
- **environment mapping**: khai báo biến lấy từ nguồn lưu trữ tương ứng.

### 🔄 Context Variations:
- **If a value is non-sensitive configuration** → Có thể dùng `variables`, nhưng vẫn không nên để cấu hình sai môi trường.
- **If the application needs runtime secret rotation** → Thiết kế runtime retrieval/rotation phù hợp; không chỉ lấy secret lúc build rồi đóng vào image.

---

## 8. Nhận diện CodeBuild execution flow

### Question 8:
A CodeBuild build must use a custom Docker image, store reusable dependency files in S3, upload build artifacts to S3, and send logs to CloudWatch Logs. Which statement is correct?

```
A. CodeBuild can run a prepackaged or custom Docker image, use an optional S3 cache, and store artifacts/logs in their configured destinations
B. CodeBuild requires an EC2 instance managed by the customer
C. S3 cache replaces the source repository
D. CloudWatch Logs is the only possible artifact destination
```

### ✅ Answer: **A. CodeBuild can run a prepackaged or custom Docker image, use an optional S3 cache, and store artifacts/logs in their configured destinations**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- CodeBuild chạy build instruction trong container, có thể dùng image dựng sẵn hoặc custom image.
- S3 cache là tùy chọn để tái sử dụng dependency; artifact và log có thể được lưu ở các đích cấu hình như S3/CloudWatch Logs.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | CodeBuild là dịch vụ managed; team không cần tự quản lý build EC2 server. |
| C | Cache chỉ lưu file có thể tái sử dụng, không thay thế source input. |
| D | Artifact có thể được upload lên S3; CloudWatch Logs dùng cho log output. |

### 📝 Vocabulary Analysis:
- **custom Docker image**: image do team xây dựng với tool/dependency riêng.
- **reusable dependency files**: file dependency có thể dùng lại giữa các build.
- **artifact destination**: nơi lưu đầu ra build.

### 🔄 Context Variations:
- **If build time increases because dependencies are downloaded every time** → Cân nhắc S3 cache, nhưng bảo đảm build vẫn tái lập khi cache trống.
- **If build cannot access a private package registry** → Kiểm tra network/VPC routing, DNS và quyền truy cập.

---

## 9. Triển khai EC2 bằng CodeDeploy

### Question 9:
A company deploys an application to EC2 instances with AWS CodeDeploy. Which file describes the deployment instructions, and what must be running on the target instances?

```
A. `buildspec.yml` and the CodeBuild container
B. `appspec.yml` and the CodeDeploy Agent
C. `cdk.json` and the CodeGuru Profiler Agent
D. `package.json` and the CodePipeline service role
```

### ✅ Answer: **B. `appspec.yml` and the CodeDeploy Agent**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- `appspec.yml` mô tả cách CodeDeploy triển khai ứng dụng.
- EC2/on-premises target phải chạy CodeDeploy Agent để nhận và thực hiện deployment lifecycle.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| A | `buildspec.yml` hướng dẫn CodeBuild build/test; container build không phải agent triển khai trên EC2. |
| C | `cdk.json` là cấu hình ứng dụng CDK và Profiler Agent thu thập profile runtime. |
| D | `package.json` quản lý package dự án; CodePipeline role không thay thế CodeDeploy Agent trên instance. |

### 📝 Vocabulary Analysis:
- **deployment instructions**: chỉ dẫn về file/hook cần triển khai.
- **target instance**: EC2/on-premises server nhận ứng dụng.
- **deployment lifecycle**: các bước trong quá trình deploy.

### 🔄 Context Variations:
- **If the deployment target is Lambda** → CodeDeploy có thể chuyển traffic giữa Lambda aliases mà không cần EC2 Agent.
- **If the target is ECS** → CodeDeploy dùng cấu hình deployment ECS, không cài CodeDeploy Agent vào EC2 theo mô hình EC2/on-premises.

---

## 10. Chọn deployment configuration cho EC2 fleet

### Question 10:
An EC2 fleet must keep the greatest possible capacity during a gradual CodeDeploy rollout. The team can accept a slower deployment. Which deployment configuration best matches the requirement?

```
A. OneAtATime
B. AllAtOnce
C. HalfAtATime
D. AllAtOnce with rollback disabled
```

### ✅ Answer: **A. OneAtATime**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- `OneAtATime` triển khai từng instance, chậm nhất nhưng gây tác động thấp nhất lên capacity theo slide.
- Trong lúc một instance được cập nhật, các instance còn lại tiếp tục phục vụ.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | `AllAtOnce` nhanh nhất nhưng triển khai đồng thời lên toàn bộ instance, downtime/tác động capacity cao nhất. |
| C | `HalfAtATime` làm giảm khoảng một nửa capacity trong mỗi lượt. |
| D | Tắt rollback không giảm mức ảnh hưởng do deploy đồng loạt. |

### 📝 Vocabulary Analysis:
- **fleet**: nhóm instance cùng phục vụ một ứng dụng.
- **deployment configuration**: quy tắc xác định số/tỷ lệ instance deploy mỗi lượt.
- **capacity**: năng lực phục vụ còn lại trong lúc cập nhật.

### 🔄 Context Variations:
- **If fastest deployment is the priority and downtime is acceptable** → `AllAtOnce`.
- **If the team can tolerate temporary 50% capacity reduction** → `HalfAtATime`.

---

## 11. Blue/green deployment cho Auto Scaling Group

### Question 11:
A team wants CodeDeploy to create a new Auto Scaling Group for a blue/green deployment, copy the existing settings, and shift traffic through a load balancer. Which requirement must be met according to the course material?

```
A. The deployment must use an Elastic Load Balancer
B. The target instances must use Cognito User Pools
C. The application must store artifacts in CodeArtifact
D. The old Auto Scaling Group must be deleted before deployment
```

### ✅ Answer: **A. The deployment must use an Elastic Load Balancer**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Slide nêu blue/green deployment cho ASG tạo ASG mới từ cấu hình cũ, cho phép chọn thời gian giữ fleet cũ và yêu cầu dùng ELB.
- Load Balancer cung cấp điểm chuyển traffic giữa môi trường.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Cognito xử lý danh tính, không phải điều kiện của ASG blue/green deployment. |
| C | CodeArtifact quản lý package dependency, không phải yêu cầu chuyển traffic. |
| D | Mục tiêu blue/green là giữ môi trường cũ trong thời gian cần thiết, không xóa nó trước deploy. |

### 📝 Vocabulary Analysis:
- **blue/green deployment**: giữ môi trường cũ và môi trường mới để chuyển traffic có kiểm soát.
- **copy existing settings**: tạo ASG mới dựa trên cấu hình của ASG cũ.
- **shift traffic**: chuyển request sang môi trường mới.

### 🔄 Context Variations:
- **If updating existing instances without a replacement group** → Dùng in-place deployment.
- **If old fleet must remain available for a defined period** → Cấu hình thời gian giữ ASG cũ trước khi dọn dẹp.

---

## 12. Chuyển traffic cho Lambda bằng CodeDeploy

### Question 12:
A SAM-managed Lambda function must publish a new version and shift traffic gradually through an alias, with a small canary percentage receiving traffic before the remainder. Which deployment mechanism should the team use?

```
A. CodeDeploy traffic shifting integrated with SAM
B. CodeArtifact upstream repositories
C. CodeBuild cache configuration
D. CodeCommit SSH authentication
```

### ✅ Answer: **A. CodeDeploy traffic shifting integrated with SAM**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- SAM tích hợp CodeDeploy cho Lambda alias traffic shifting.
- Canary đưa một tỷ lệ nhỏ vào version mới trước rồi mới chuyển toàn bộ traffic; alarm có thể hỗ trợ rollback.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Upstream repository lưu/truy xuất dependency, không đổi tỷ lệ traffic Lambda. |
| C | Cache CodeBuild tối ưu vòng lặp build, không điều phối runtime traffic. |
| D | SSH authentication xác thực Git client tới repository. |

### 📝 Vocabulary Analysis:
- **alias**: tên ổn định trỏ tới một Lambda version.
- **canary percentage**: tỷ lệ request thử nghiệm trên version mới.
- **remainder**: phần traffic còn lại sau bước canary.

### 🔄 Context Variations:
- **If traffic should increase in equal steps over time** → Chọn Linear deployment preference.
- **If all traffic can move immediately** → `AllAtOnce`, nhưng không có giai đoạn canary.

---

## 13. Nhận diện chiến lược CodeDeploy cho ECS

### Question 13:
A team uses CodeDeploy to deploy a new ECS task definition. Which deployment strategy is supported according to the course material?

```
A. Blue/green only, with options such as canary, linear, or all-at-once traffic shifting
B. In-place only with the CodeDeploy Agent installed in every container
C. OneAtATime only
D. Direct deployment to an S3 bucket
```

### ✅ Answer: **A. Blue/green only, with options such as canary, linear, or all-at-once traffic shifting**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Slide nêu CodeDeploy tự động triển khai ECS task definition mới theo blue/green.
- Có thể chọn traffic-shifting strategy Linear, Canary hoặc AllAtOnce.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | In-place và CodeDeploy Agent trên host thuộc mô hình EC2/on-premises; slide mô tả ECS dùng blue/green. |
| C | `OneAtATime` là deployment configuration cho fleet EC2/on-premises trong slide. |
| D | S3 có thể là đích deploy của một số pipeline action, nhưng không mô tả CodeDeploy triển khai ECS task definition. |

### 📝 Vocabulary Analysis:
- **task definition**: định nghĩa cấu hình container/task của ECS.
- **blue/green only**: chỉ dùng mô hình triển khai hai môi trường xanh/xanh dương.
- **traffic shifting**: chuyển dần hoặc chuyển ngay request sang phiên bản mới.

### 🔄 Context Variations:
- **If the target is Lambda** → CodeDeploy chuyển traffic giữa Lambda aliases.
- **If the target is EC2/on-premises** → Có thể chọn in-place hoặc blue/green.

---

## 14. Hiểu rollback của CodeDeploy

### Question 14:
A CodeDeploy deployment fails and an automatic rollback starts. What does CodeDeploy do according to the course material?

```
A. Creates a new deployment using the last known good application revision
B. Restores the exact old deployment object without running a deployment
C. Reverts all database changes automatically
D. Rebuilds the application from the latest source commit
```

### ✅ Answer: **A. Creates a new deployment using the last known good application revision**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Rollback của CodeDeploy redeploy revision tốt gần nhất như một deployment mới.
- Đây không phải thao tác phục hồi nguyên trạng deployment cũ.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Slide nêu CodeDeploy tạo deployment mới từ revision trước đó, không restore một deployment object cũ. |
| C | CodeDeploy triển khai ứng dụng; không tự đảo ngược mọi migration hoặc side effect của database. |
| D | Rollback dùng revision tốt đã triển khai, không mặc định build từ commit mới nhất. |

### 📝 Vocabulary Analysis:
- **last known good revision**: revision gần nhất đã được xác định là hoạt động tốt.
- **automatic rollback**: tự động bắt đầu deploy lại revision trước theo điều kiện cấu hình.
- **side effect**: thay đổi dữ liệu/tác động ngoài việc thay file ứng dụng.

### 🔄 Context Variations:
- **If a database migration is not backward-compatible** → Cần kế hoạch migration/compensation riêng; CodeDeploy rollback không tự khôi phục dữ liệu.
- **If rollback is disabled** → Deployment không tự thực hiện rollback theo cấu hình đó.

---

## 15. Quản lý package dependency

### Question 15:
Developers and CodeBuild need a managed repository for packages used by Maven, npm, pip, and NuGet projects. Which AWS service is designed for this purpose?

```
A. AWS CodeArtifact
B. AWS CodeDeploy
C. Amazon CodeGuru Reviewer
D. AWS CodeCommit
```

### ✅ Answer: **A. AWS CodeArtifact**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- CodeArtifact quản lý software package/dependency và tích hợp với Maven, npm, pip, NuGet cùng package manager khác.
- Developer và CodeBuild có thể lấy dependency từ repository tập trung.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | CodeDeploy triển khai application revision, không quản lý dependency package. |
| C | Reviewer phân tích mã tĩnh và đề xuất cải thiện, không host package repository. |
| D | CodeCommit host Git source repository, không phải package artifact repository. |

### 📝 Vocabulary Analysis:
- **dependency**: package/thư viện mà phần mềm cần để build hoặc chạy.
- **artifact management**: lưu trữ và phân phối package được tạo hoặc sử dụng trong phát triển.
- **managed repository**: repository do dịch vụ quản lý.

### 🔄 Context Variations:
- **If the requirement is to version application source code** → Dùng Git provider như CodeCommit/GitHub.
- **If a package version is published and should trigger checks** → Tích hợp CodeArtifact với EventBridge.

---

## 16. Phản ứng với package event

### Question 16:
A new package version in CodeArtifact should start an automated security validation and then rebuild and redeploy an application. Which integration can route CodeArtifact package events to these actions?

```
A. Amazon EventBridge
B. AWS CodeGuru Profiler
C. CodeDeploy Agent only
D. Cognito Identity Pool
```

### ✅ Answer: **A. Amazon EventBridge**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- CodeArtifact phát event khi package version được tạo, sửa hoặc xóa.
- EventBridge có thể route event đến Lambda, Step Functions, SNS, SQS hoặc khởi chạy CodePipeline để build/redeploy.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Profiler theo dõi runtime behavior, không định tuyến package event. |
| C | Agent thực hiện deployment trên target, không nhận event CodeArtifact để điều phối validation pipeline. |
| D | Identity Pool cấp AWS credentials, không route sự kiện package. |

### 📝 Vocabulary Analysis:
- **package version event**: sự kiện phát sinh khi version package được tạo/sửa/xóa.
- **route an event**: định tuyến event đến target xử lý.
- **security validation**: bước kiểm tra bảo mật trước khi phát hành.

### 🔄 Context Variations:
- **If the event should notify a team only** → Route tới SNS/SQS hoặc Lambda gửi thông báo.
- **If the event should execute multi-step remediation** → Route tới Step Functions hoặc CodePipeline phù hợp.

---

## 17. Phân biệt CodeGuru Reviewer và Profiler

### Question 17:
A Java application has high CPU usage in production, and the team wants to identify which runtime code path consumes excessive CPU. Which CodeGuru capability should the team use?

```
A. CodeGuru Profiler
B. CodeGuru Reviewer
C. CodeArtifact
D. CodePipeline manual approval
```

### ✅ Answer: **A. CodeGuru Profiler**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Profiler quan sát runtime behavior và giúp tìm routine/code path tiêu thụ CPU hoặc memory quá mức.
- Đây là câu hỏi hiệu năng production, không phải lỗi code tĩnh trong pull request.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Reviewer phân tích code tĩnh để tìm issue/security/best practice trong development. |
| C | CodeArtifact lưu dependency/package. |
| D | Manual approval kiểm soát pipeline release, không profile CPU. |

### 📝 Vocabulary Analysis:
- **runtime code path**: đường thực thi code khi ứng dụng đang chạy.
- **excessive CPU**: mức dùng CPU quá cao so với mục tiêu.
- **production**: môi trường đang phục vụ người dùng thật.

### 🔄 Context Variations:
- **If the team wants static review for resource leaks or input validation in Java/Python** → CodeGuru Reviewer.
- **If the team wants to inspect heap usage or anomalies at runtime** → CodeGuru Profiler.
