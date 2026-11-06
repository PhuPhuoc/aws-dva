# Câu hỏi ôn tập DVA-C02 - Section 14: AWS Serverless, SAM, CDK, Cognito và các dịch vụ liên quan

> *Câu hỏi và lựa chọn bằng tiếng Anh; phần giải thích bằng tiếng Việt.*

---

## 1. Nhận diện SAM template

### Question 1:
A developer wants to define a Lambda function and API Gateway endpoint in a concise YAML template, then deploy them through CloudFormation. Which framework should the developer use?

```
A. AWS SAM
B. AWS CDK only
C. Amazon Cognito Identity Pools
D. AWS AppSync
```

### ✅ Answer: **A. AWS SAM**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- SAM là framework serverless dùng template YAML rút gọn cho các tài nguyên như `AWS::Serverless::Function` và `AWS::Serverless::Api`.
- SAM transform template thành CloudFormation để quản lý triển khai.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | CDK định nghĩa hạ tầng bằng ngôn ngữ lập trình rồi synth thành CloudFormation; đề bài mô tả SAM template ngắn gọn bằng YAML. |
| C | Identity Pool cấp temporary AWS credentials cho danh tính federated, không định nghĩa Lambda/API. |
| D | AppSync cung cấp GraphQL API managed, không phải framework template SAM. |

### 📝 Vocabulary Analysis:
- **concise**: ngắn gọn.
- **deploy through CloudFormation**: triển khai và quản lý tài nguyên qua CloudFormation.
- **framework**: bộ công cụ/quy ước để xây dựng ứng dụng.

### 🔄 Context Variations:
- **If the team wants to define infrastructure using TypeScript or Python** → Xem xét AWS CDK.
- **If the API requirement is GraphQL with resolvers and subscriptions** → Xem xét AWS AppSync.

---

## 2. Chọn cách đồng bộ mã SAM trong phát triển

### Question 2:
A developer is iterating on Lambda code in a development environment. The infrastructure has not changed, and the developer wants the fastest SAM workflow that updates code without using CloudFormation. Which command should the developer run?

```
A. sam local generate-event
B. sam sync --code
C. cdk bootstrap
D. sam local start-api
```

### ✅ Answer: **B. `sam sync --code`**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- `sam sync --code` đồng bộ thay đổi mã mà không cập nhật hạ tầng; slide mô tả lệnh này bỏ qua CloudFormation để rút ngắn thời gian cập nhật.
- Đây là lựa chọn phù hợp với vòng lặp phát triển nhanh khi cấu hình hạ tầng không thay đổi.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| A | `sam local generate-event` tạo payload mẫu, không đồng bộ code lên AWS. |
| C | `cdk bootstrap` chuẩn bị tài nguyên CDK Toolkit cho một AWS environment. |
| D | `sam local start-api` khởi động API cục bộ, không cập nhật code trên AWS. |

### 📝 Vocabulary Analysis:
- **iterating on code**: chỉnh sửa và thử mã lặp nhanh.
- **infrastructure has not changed**: cấu hình hạ tầng không thay đổi.
- **bypass CloudFormation**: bỏ qua CloudFormation trong lần cập nhật đó.

### 🔄 Context Variations:
- **If the template or infrastructure configuration changed** → Dùng luồng `sam sync` phù hợp thay vì chỉ `--code`.
- **If the team needs a tracked production release** → Dùng quy trình triển khai quản lý stack như `sam deploy`/pipeline CloudFormation.

---

## 3. SAM và triển khai Lambda an toàn

### Question 3:
A team deploys a Lambda function with AWS SAM. It needs to publish a new function version, move a small percentage of traffic to the new version first, and automatically roll back if a CloudWatch alarm indicates errors. Which configuration is most appropriate?

```
A. AutoPublishAlias with a Canary DeploymentPreference and an alarm
B. sam local invoke with a production payload
C. A Cognito Identity Pool with guest access
D. An API Gateway usage plan
```

### ✅ Answer: **A. AutoPublishAlias with a Canary DeploymentPreference and an alarm**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- `AutoPublishAlias` tạo version mới và chuyển alias sang version đó.
- `DeploymentPreference` hỗ trợ Canary/Linear/AllAtOnce; alarm có thể kích hoạt rollback nếu bản mới có dấu hiệu lỗi.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | `sam local invoke` dùng để gọi function cục bộ một lần, không điều phối traffic production. |
| C | Identity Pool cấp AWS credentials cho người dùng, không triển khai version Lambda. |
| D | Usage plan giới hạn/đo lường request API, không chuyển traffic giữa Lambda versions. |

### 📝 Vocabulary Analysis:
- **a small percentage of traffic**: một tỷ lệ nhỏ lưu lượng.
- **roll back**: quay lui về phiên bản triển khai trước đó.
- **alarm indicates errors**: alarm báo có lỗi vượt ngưỡng.

### 🔄 Context Variations:
- **If all traffic should move immediately** → `AllAtOnce` có thể phù hợp, nhưng không có giai đoạn quan sát traffic nhỏ trước.
- **If the team needs to validate behavior before or after shifting traffic** → Thêm pre-traffic/post-traffic hooks.

---

## 4. Kiểm thử API cục bộ với SAM

### Question 4:
A developer wants to start a local HTTP server that exposes API Gateway routes defined in a SAM template and automatically reloads changed function code. Which command should the developer use?

```
A. sam local start-api
B. sam local invoke
C. sam package
D. cdk diff
```

### ✅ Answer: **A. `sam local start-api`**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- `sam local start-api` khởi động HTTP server cục bộ cho các function được khai báo và tự nạp lại khi mã thay đổi.
- Đây là cách kiểm tra route và handler trong vòng lặp phát triển local.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | `sam local invoke` gọi một function với payload một lần rồi kết thúc; không host API endpoint. |
| C | `sam package` chuẩn bị/đóng gói artifact cho triển khai, không mở server local. |
| D | `cdk diff` xem thay đổi giữa template CDK và stack đã triển khai. |

### 📝 Vocabulary Analysis:
- **exposes API routes**: cung cấp các đường dẫn API.
- **automatically reloads**: tự nạp lại khi có thay đổi.
- **handler**: hàm xử lý invocation/request.

### 🔄 Context Variations:
- **If the developer wants a single invocation with a sample event** → Dùng `sam local invoke`.
- **If the developer wants to generate a sample event payload** → Dùng `sam local generate-event`.

---

## 5. Chọn giữa SAM và CDK

### Question 5:
A team wants to define infrastructure in TypeScript, reuse higher-level constructs, and generate a CloudFormation template before deployment. Which tool best matches these requirements?

```
A. AWS SAM
B. AWS CDK
C. Amazon Cognito User Pools
D. AWS Secrets Manager
```

### ✅ Answer: **B. AWS CDK**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- CDK cho phép định nghĩa hạ tầng bằng TypeScript và các ngôn ngữ lập trình được hỗ trợ.
- Construct giúp tái sử dụng abstraction; `cdk synth` tổng hợp mã thành CloudFormation template.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| A | SAM tập trung serverless và khai báo template bằng YAML/JSON, không dùng TypeScript làm ngôn ngữ định nghĩa template SAM. |
| C | Cognito User Pool quản lý người dùng và đăng nhập ứng dụng. |
| D | Secrets Manager lưu trữ và quản lý secret. |

### 📝 Vocabulary Analysis:
- **higher-level construct**: thành phần trừu tượng cấp cao có thể tái sử dụng.
- **generate a template**: tổng hợp thành template triển khai.
- **best matches**: phù hợp nhất với yêu cầu.

### 🔄 Context Variations:
- **If the team wants a concise YAML template focused on Lambda/API Gateway** → SAM thường phù hợp hơn.
- **If the team needs a programming language and abstractions across many AWS services** → CDK phù hợp hơn.

---

## 6. Chuẩn bị AWS environment cho CDK

### Question 6:
A developer deploys a CDK application to a new AWS account and Region. The deployment fails because the CDK Toolkit resources are not present. What should the developer do first?

```
A. Run `cdk bootstrap aws://<account>/<region>` for that environment
B. Run `sam local start-lambda`
C. Create a Cognito User Pool
D. Run `cdk destroy`
```

### ✅ Answer: **A. Run `cdk bootstrap aws://<account>/<region>` for that environment**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- CDK bootstrap tạo CloudFormation stack `CDKToolkit` và tài nguyên staging cần thiết như S3 bucket và IAM Role.
- Mỗi account/Region mới cần được bootstrap trước khi triển khai CDK vào environment đó.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Lệnh SAM chạy Lambda cục bộ, không tạo tài nguyên cho CDK deployment. |
| C | User Pool không phải tài nguyên bootstrap của CDK. |
| D | `cdk destroy` xóa stack, không chuẩn bị môi trường triển khai. |

### 📝 Vocabulary Analysis:
- **new AWS environment**: account và Region đích chưa được chuẩn bị cho CDK.
- **Toolkit resources**: tài nguyên phục vụ quá trình triển khai CDK.
- **bootstrap**: cấp phát trước hạ tầng cần cho deployment.

### 🔄 Context Variations:
- **If the target account/Region was already bootstrapped** → Review role/permission và `cdk diff` thay vì chạy destroy.
- **If the deployment is a SAM application** → CDK bootstrap không phải bước triển khai SAM.

---

## 7. Chọn loại kiểm thử CDK

### Question 7:
A team wants a CDK test that checks whether a generated IAM policy contains a specific action and resource. Which testing approach is most suitable?

```
A. Fine-grained CDK assertion
B. Snapshot test only
C. `sam local generate-event`
D. AppSync subscription
```

### ✅ Answer: **A. Fine-grained CDK assertion**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Fine-grained assertion kiểm tra một thuộc tính cụ thể của CloudFormation template, như action/resource trong policy.
- Assertion thể hiện trực tiếp điều kiện cần giữ và ít nhiễu hơn khi phần không liên quan của template thay đổi.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Snapshot có thể phát hiện thay đổi toàn template nhưng không diễn đạt rõ invariant cụ thể bằng assertion mục tiêu. |
| C | Lệnh này tạo event mẫu cho Lambda, không kiểm thử template CDK. |
| D | Subscription là khả năng realtime của GraphQL API, không liên quan kiểm thử CloudFormation. |

### 📝 Vocabulary Analysis:
- **generated policy**: IAM policy được tổng hợp từ mã CDK.
- **specific action and resource**: action và tài nguyên cụ thể cần xác minh.
- **invariant**: điều kiện phải luôn đúng.

### 🔄 Context Variations:
- **If the team wants to compare the entire synthesized template with a stored baseline** → Snapshot test phù hợp, nhưng nên bổ sung assertions cho các quy tắc quan trọng.
- **If the team is validating runtime behavior** → Cần integration test; assertion template chỉ kiểm tra cấu hình được tổng hợp.

---

## 8. Phân biệt Cognito User Pool và Identity Pool

### Question 8:
A mobile application needs username/password sign-in, password reset, MFA, and JWTs for its users. It does not need the mobile client to call AWS services directly. Which Cognito component should the application use?

```
A. Cognito User Pool
B. Cognito Identity Pool only
C. AWS CDK bootstrap
D. AWS Step Functions Activity
```

### ✅ Answer: **A. Cognito User Pool**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- User Pool quản lý người dùng ứng dụng và tính năng đăng nhập như password reset, MFA, liên kết danh tính; sau đăng nhập trả JWT.
- Yêu cầu không cần client nhận AWS credentials để gọi dịch vụ AWS trực tiếp.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Identity Pool chủ yếu đổi danh tính federated thành temporary AWS credentials; không phải lựa chọn chính cho user directory và đăng nhập username/password. |
| C | Bootstrap chuẩn bị tài nguyên CDK. |
| D | Activity là worker task trong Step Functions, không quản lý người dùng. |

### 📝 Vocabulary Analysis:
- **password reset**: đặt lại mật khẩu.
- **JWT**: JSON Web Token mang claims/danh tính đã được xác thực.
- **call AWS services directly**: client gọi dịch vụ AWS bằng AWS credentials.

### 🔄 Context Variations:
- **If the mobile client must upload directly to S3 with temporary credentials** → Kết hợp User Pool với Identity Pool và IAM Role giới hạn.
- **If the application needs federated sign-in through OIDC/SAML** → Có thể cấu hình IdP tương ứng với User Pool hoặc Identity Pool tùy luồng cần dùng.

---

## 9. Cấp temporary AWS credentials cho người dùng

### Question 9:
Users authenticate with a social identity provider. The mobile application must receive temporary AWS credentials so it can access a specific S3 prefix directly. Which service should map the federated identities to an IAM role?

```
A. Cognito Identity Pool
B. Cognito User Pool only
C. AWS AppSync schema
D. SAM policy template
```

### ✅ Answer: **A. Cognito Identity Pool**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Identity Pool nhận danh tính từ social provider và cấp temporary AWS credentials thông qua IAM Role.
- Role policy có thể giới hạn quyền vào đúng S3 prefix cần thiết.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | User Pool quản lý đăng nhập/tokens; riêng User Pool không phải cơ chế cấp AWS credentials trực tiếp cho client. |
| C | AppSync schema định nghĩa hợp đồng GraphQL, không ánh xạ danh tính sang IAM Role. |
| D | SAM policy template khai báo quyền cho Lambda function, không cấp credentials cho người dùng mobile. |

### 📝 Vocabulary Analysis:
- **federated identity**: danh tính được xác thực bởi một identity provider khác.
- **temporary AWS credentials**: credentials AWS có thời hạn.
- **specific S3 prefix**: một nhánh/key prefix cụ thể trong bucket S3.

### 🔄 Context Variations:
- **If users only need to authenticate to the application and receive JWTs** → User Pool có thể đáp ứng mà không cần Identity Pool.
- **If a backend Lambda accesses S3 on behalf of users** → Có thể giữ credentials ở backend và cấp IAM Role cho Lambda; không nhất thiết đưa AWS credentials xuống client.

---

## 10. Tách xác thực và phân quyền trong ứng dụng Cognito

### Question 10:
An API validates a Cognito User Pool JWT successfully. The authenticated user requests an order record owned by another user. Which additional control should the application implement?

```
A. Verify authorization for the requested order in the backend
B. Trust every valid JWT to access every order
C. Enable a Cognito Identity Pool with unrestricted permissions
D. Replace the JWT with an API key
```

### ✅ Answer: **A. Verify authorization for the requested order in the backend**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- User Pool xác minh danh tính/token, nhưng token hợp lệ không có nghĩa user được phép truy cập mọi order.
- Backend phải kiểm tra ownership hoặc policy nghiệp vụ cho resource đang được yêu cầu.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Chỉ xác thực danh tính mà bỏ kiểm tra quyền sẽ tạo lỗ hổng truy cập ngang giữa người dùng. |
| C | Quyền rộng trong Identity Pool không giải quyết kiểm tra quyền theo từng order và làm tăng rủi ro. |
| D | API key không thay thế xác thực user và authorization theo resource. |

### 📝 Vocabulary Analysis:
- **authenticated user**: người dùng đã được xác minh danh tính.
- **owned by another user**: thuộc sở hữu của người dùng khác.
- **authorization**: quyết định principal được phép thực hiện hành động nào trên resource.

### 🔄 Context Variations:
- **If the order is owned by the caller** → Backend vẫn xác minh owner/tenant trước khi trả dữ liệu.
- **If the API uses Cognito Identity Pool credentials** → IAM policy có thể giảm phạm vi quyền AWS, nhưng kiểm tra quyền nghiệp vụ ở backend vẫn cần thiết khi phù hợp.

---

## 11. Xử lý lỗi trong Step Functions

### Question 11:
A Step Functions task calls a downstream service that may fail temporarily. The workflow should retry selected transient errors with a delay and send the task to a recovery state after retries are exhausted. Which feature should be configured?

```
A. Retry and Catch on the Task state
B. A Pass state only
C. An Amplify Hosting redirect
D. A Cognito hosted UI trigger
```

### ✅ Answer: **A. Retry and Catch on the Task state**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- `Retry` khớp loại lỗi, đặt thời gian chờ/backoff và giới hạn số lần thử.
- Sau khi hết retry, `Catch` có thể chuyển workflow sang state recovery bằng `Next` và giữ dữ liệu lỗi qua `ResultPath`.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Pass chỉ chuyển/đưa dữ liệu vào luồng, không tự thiết lập chính sách retry và recovery. |
| C | Redirect của Amplify Hosting thuộc luồng phục vụ frontend. |
| D | Hosted UI trigger liên quan đăng nhập Cognito, không xử lý lỗi Task trong workflow. |

### 📝 Vocabulary Analysis:
- **transient error**: lỗi tạm thời có thể hết khi thử lại.
- **retries are exhausted**: đã dùng hết số lần retry được cấu hình.
- **recovery state**: state xử lý tiếp sau khi một bước thất bại.

### 🔄 Context Variations:
- **If the failure is permanent, such as invalid input** → Không retry lặp lại; Catch để báo lỗi hoặc xử lý input.
- **If the workflow should stop immediately on success/failure** → Dùng Succeed/Fail state sau nhánh xử lý tương ứng.

---

## 12. Chờ phê duyệt bên ngoài trong Step Functions

### Question 12:
An order workflow must pause while a human approves a payment. An external application will call Step Functions after the decision. Which integration pattern should the workflow use?

```
A. A Task using `.waitForTaskToken`, resumed with SendTaskSuccess or SendTaskFailure
B. An Express synchronous workflow that never waits
C. A Choice state that polls a user interface
D. A Cognito User Pool JWT as the workflow result
```

### ✅ Answer: **A. A Task using `.waitForTaskToken`, resumed with SendTaskSuccess or SendTaskFailure**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Task Token pattern cho phép state machine tạm dừng và chờ hệ thống bên ngoài gửi callback.
- Quy trình được tiếp tục bằng `SendTaskSuccess` hoặc `SendTaskFailure` kèm token.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Express synchronous chờ workflow hoàn tất trong request, không phù hợp với phê duyệt có thể kéo dài. |
| C | Choice đánh giá dữ liệu/điều kiện trong execution; không tự kết nối và poll UI người dùng. |
| D | JWT là token danh tính, không phải callback token để tiếp tục Task. |

### 📝 Vocabulary Analysis:
- **pause**: tạm dừng thực thi.
- **external application**: ứng dụng/hệ thống bên ngoài state machine.
- **resumed**: được tiếp tục sau khi callback đến.

### 🔄 Context Variations:
- **If a worker should poll for work instead of receiving a callback** → Dùng Activity Task với `GetActivityTask` và heartbeat.
- **If the external system reports a failed approval** → Gọi `SendTaskFailure` để workflow xử lý nhánh lỗi.

---

## 13. Chọn Standard hay Express workflow

### Question 13:
A system processes a high volume of short-lived events asynchronously. It does not need an immediate response, and the application can make processing idempotent. Which Step Functions workflow type is the best fit?

```
A. Standard workflow
B. Express asynchronous workflow
C. Express synchronous workflow only
D. Cognito Identity Pool
```

### ✅ Answer: **B. Express asynchronous workflow**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Express phù hợp workflow ngắn, thông lượng cao; biến thể asynchronous không chờ hoàn tất và theo slide có mô hình at-least-once.
- Đề bài nói rõ không cần response ngay và ứng dụng có thể xử lý idempotency, phù hợp với đặc tính này.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| A | Standard phù hợp workflow dài/những trường hợp cần lịch sử execution; không phải lựa chọn nhắm tới high-volume short-lived processing trong đề. |
| C | Express synchronous chờ workflow hoàn tất, trong khi yêu cầu là asynchronous và không cần response ngay. |
| D | Identity Pool cấp AWS credentials, không điều phối workflow. |

### 📝 Vocabulary Analysis:
- **high volume**: lưu lượng hoặc số lượng sự kiện lớn.
- **short-lived**: có thời gian chạy ngắn.
- **idempotent**: gọi lặp lại vẫn không gây thêm tác động ngoài kết quả mong muốn.
- **asynchronously**: xử lý bất đồng bộ, caller không đợi kết quả hoàn tất.

### 🔄 Context Variations:
- **If the workflow can run for months or needs long execution history** → Xem xét Standard.
- **If the caller must wait for the result immediately** → Xem xét Express synchronous nếu thời lượng và semantics phù hợp.

---

## 14. Chọn API cho GraphQL và nhiều data source

### Question 14:
A mobile application needs a managed GraphQL API that can retrieve selected fields from DynamoDB and combine them with data from a Lambda data source. It also needs real-time updates. Which service should the team use?

```
A. AWS AppSync
B. AWS SAM Local
C. CodeDeploy
D. Cognito Identity Pool only
```

### ✅ Answer: **A. AWS AppSync**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- AppSync cung cấp GraphQL API managed, kết nối được với DynamoDB và custom data source như Lambda.
- AppSync hỗ trợ subscription/realtime qua WebSocket; ứng dụng mobile cũng có thể dùng tính năng local data synchronization.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | SAM Local mô phỏng một phần Lambda/API Gateway trên máy phát triển, không phải dịch vụ GraphQL managed. |
| C | CodeDeploy tự động hóa triển khai, không tổng hợp dữ liệu GraphQL. |
| D | Identity Pool cấp credentials cho người dùng; không định nghĩa schema hoặc resolver GraphQL. |

### 📝 Vocabulary Analysis:
- **selected fields**: các trường client yêu cầu trong response.
- **data source**: nguồn dữ liệu mà resolver truy vấn.
- **real-time updates**: cập nhật dữ liệu gần thời gian thực.

### 🔄 Context Variations:
- **If clients need only a REST/HTTP proxy to one backend** → API Gateway có thể phù hợp hơn.
- **If the GraphQL API needs Cognito user authorization** → AppSync hỗ trợ `AMAZON_COGNITO_USER_POOLS` authorization mode.

---

## 15. Chọn authorization mode cho AppSync

### Question 15:
A GraphQL API must authorize signed AWS workloads using IAM roles, while another API may use Cognito User Pool tokens. Which statement about AppSync authorization is correct?

```
A. AppSync supports AWS_IAM and AMAZON_COGNITO_USER_POOLS authorization modes
B. AppSync supports only API_KEY authentication
C. AppSync requires every request to use a Cognito Identity Pool
D. AppSync cannot integrate with OIDC providers
```

### ✅ Answer: **A. AppSync supports AWS_IAM and AMAZON_COGNITO_USER_POOLS authorization modes**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- AppSync hỗ trợ nhiều mode gồm `AWS_IAM`, `AMAZON_COGNITO_USER_POOLS`, `API_KEY` và `OPENID_CONNECT`.
- IAM phù hợp workload dùng AWS identity; Cognito User Pools phù hợp token người dùng ứng dụng.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | API key chỉ là một trong nhiều authorization mode của AppSync. |
| C | AppSync không bắt buộc mọi request phải qua Identity Pool; có thể dùng các mode khác. |
| D | `OPENID_CONNECT` là một trong các mode được hỗ trợ. |

### 📝 Vocabulary Analysis:
- **signed AWS workload**: workload ký request bằng thông tin xác thực AWS, thường qua SigV4.
- **authorization mode**: cơ chế AppSync dùng để xác minh request được phép truy cập API.
- **OIDC provider**: nhà cung cấp danh tính tương thích OpenID Connect.

### 🔄 Context Variations:
- **If the API is public and uses a key for a limited use case** → `API_KEY` có thể phù hợp, nhưng không nên coi key là xác thực mạnh cho dữ liệu riêng tư.
- **If users authenticate through an external OIDC provider** → Có thể chọn `OPENID_CONNECT`.

---

## 16. Nhận diện tính năng AWS Amplify

### Question 16:
A frontend team wants to build and host a modern web application, run CI/CD builds, preview pull requests, configure a custom domain, and run Cypress end-to-end tests during the build phase. Which service best fits this workflow?

```
A. AWS Amplify
B. AWS Step Functions Standard
C. Cognito Identity Pools
D. AWS CDK Assertions
```

### ✅ Answer: **A. AWS Amplify**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Amplify Hosting hỗ trợ build/host ứng dụng web, CI/CD, pull request preview, custom domain và monitoring.
- Amplify có thể chạy E2E test trong build phase và tích hợp Cypress theo nội dung slide.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Step Functions điều phối workflow backend, không host frontend hoặc chạy preview pull request. |
| C | Identity Pool cấp AWS credentials cho người dùng, không cung cấp quy trình build/hosting frontend. |
| D | CDK Assertions kiểm thử CloudFormation template, không kiểm thử UI end-to-end. |

### 📝 Vocabulary Analysis:
- **pull request preview**: bản preview được tạo cho một pull request.
- **build phase**: giai đoạn build trong pipeline.
- **end-to-end test**: kiểm thử toàn luồng từ giao diện đến các dependency.

### 🔄 Context Variations:
- **If the requirement is user sign-up and sign-in** → Amplify có thể tích hợp Cognito cho Authentication.
- **If the requirement is local data synchronization backed by GraphQL** → Xem xét Amplify DataStore cùng AppSync/DynamoDB.

---

## 17. Phân biệt SAM policy template và Cognito user permissions

### Question 17:
A Lambda function defined in a SAM template must poll one SQS queue. Which option is the most directly relevant permission shortcut to inspect in the SAM policy templates?

```
A. SQSPollerPolicy
B. S3ReadPolicy
C. Cognito User Pool hosted UI
D. AppSync OPENID_CONNECT mode
```

### ✅ Answer: **A. `SQSPollerPolicy`**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- `SQSPollerPolicy` là policy template được slide nêu cho phép Lambda poll SQS queue.
- Sau khi áp dụng template, vẫn cần kiểm tra resource scope và quyền thực tế để bảo đảm least privilege.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | `S3ReadPolicy` cấp quyền chỉ đọc object trong S3, không poll SQS. |
| C | Hosted UI phục vụ sign-up/sign-in người dùng, không cấp quyền cho Lambda đọc queue. |
| D | `OPENID_CONNECT` là authorization mode của AppSync, không phải policy template SAM cho SQS. |

### 📝 Vocabulary Analysis:
- **poll a queue**: định kỳ yêu cầu/đọc message từ queue.
- **permission shortcut**: mẫu khai báo quyền được cấu hình sẵn.
- **resource scope**: phạm vi tài nguyên mà policy áp dụng.

### 🔄 Context Variations:
- **If the Lambda needs read-only access to S3 objects** → Xem xét `S3ReadPolicy` và giới hạn bucket/prefix.
- **If the function needs DynamoDB create/read/update/delete permissions** → Xem xét `DynamoDBCrudPolicy` và thu hẹp resource phù hợp.
