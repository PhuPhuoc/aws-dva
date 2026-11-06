# Câu hỏi ôn tập DVA-C02 - Section 16: Identity, Security và dịch vụ AWS

> *Câu hỏi và lựa chọn bằng tiếng Anh; phần giải thích bằng tiếng Việt.*

---

## 1. Dùng STS thay cho credentials dài hạn

### Question 1:
An application running on an EC2 instance needs to call Amazon S3. The team wants **temporary credentials** and does not want to store long-term access keys on the instance. What should the team use?

```
A. An IAM role attached to the EC2 instance
B. The AWS account root access key
C. A shared IAM user access key in a configuration file
D. A public S3 bucket policy
```

### ✅ Answer: **A. An IAM role attached to the EC2 instance**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- EC2 instance nhận temporary credentials thông qua IAM Role; ứng dụng dùng credentials này để gọi S3 mà không lưu access key dài hạn.
- Role policy có thể giới hạn vào đúng bucket/action cần dùng.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Root key có quyền quá rộng và không nên dùng cho ứng dụng. |
| C | Key dài hạn trong file có thể bị lộ, khó rotate và audit. |
| D | Public bucket policy làm dữ liệu công khai, không cấp credentials an toàn cho instance. |

### 📝 Vocabulary Analysis:
- **temporary credentials**: credentials có thời hạn, tự hết hiệu lực.
- **long-term access key**: access key dài hạn gắn với IAM user.
- **attached role**: IAM Role được gắn vào AWS compute resource.

### 🔄 Context Variations:
- **If the caller is a Lambda function** → Gắn execution role riêng cho Lambda.
- **If a user needs access to another account** → Dùng STS `AssumeRole` với trust policy phù hợp.

---

## 2. Cho phép cross-account AssumeRole

### Question 2:
A developer in Account A needs to assume a role in Account B. Which configuration is required for the cross-account role assumption?

```
A. Account B trusts Account A, and the developer is allowed to call sts:AssumeRole on the role
B. Account A makes the role's trust policy public
C. The developer copies an administrator's access key from Account B
D. Account B creates only an S3 bucket policy
```

### ✅ Answer: **A. Account B trusts Account A, and the developer is allowed to call sts:AssumeRole on the role**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Trust policy trong Account B xác định principal/account nào được assume role.
- Principal ở Account A cũng cần được phép gọi `sts:AssumeRole` trên ARN của role đích.
- Sau khi assume, quyền làm việc được quyết định bởi permissions policy của role.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Trust policy phải giới hạn principal, không mở công khai. |
| C | Chia sẻ access key là không an toàn và không phải cơ chế federation. |
| D | S3 bucket policy không tạo quan hệ AssumeRole cho IAM role. |

### 📝 Vocabulary Analysis:
- **trust policy**: policy quy định ai được assume role.
- **permissions policy**: policy quy định role được làm gì.
- **role assumption**: STS cấp session credentials của role cho principal được tin cậy.

### 🔄 Context Variations:
- **If the target is an S3 bucket shared directly across accounts** → Cần principal-side identity permission và bucket policy cho phép truy cập.
- **If a third-party vendor assumes the role** → Có thể cần cấu hình `ExternalId` theo mô hình tin cậy.

---

## 3. Chọn cơ chế cho đăng nhập OIDC của ứng dụng

### Question 3:
A mobile application accepts sign-in from an external OIDC identity provider. The application needs temporary AWS credentials for its users. Which approach matches the course recommendation?

```
A. Use Amazon Cognito Identity Pools to exchange the identity for AWS credentials
B. Embed an IAM user's access key in the mobile application
C. Use the AWS account root credentials
D. Store the OIDC token in an S3 object and make it public
```

### ✅ Answer: **A. Use Amazon Cognito Identity Pools to exchange the identity for AWS credentials**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Cognito Identity Pools hỗ trợ liên kết identity provider với quyền AWS tạm thời.
- Slide khuyến nghị dùng Cognito Identity Pools thay vì ứng dụng tự gọi `AssumeRoleWithWebIdentity` trực tiếp theo kiểu cũ.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Mobile app có thể bị reverse-engineer; access key dài hạn không thể giữ kín. |
| C | Root credential không được nhúng vào ứng dụng và có quyền vượt nhu cầu. |
| D | Token công khai làm lộ bearer credential và không tạo kiểm soát IAM đúng cách. |

### 📝 Vocabulary Analysis:
- **identity provider (IdP)**: dịch vụ xác thực danh tính, ví dụ OIDC provider.
- **exchange the identity**: đổi identity token thành AWS credentials tạm thời.
- **reverse-engineer**: phân tích ứng dụng để lấy thông tin nhúng bên trong.

### 🔄 Context Variations:
- **If employees use corporate SAML federation** → `AssumeRoleWithSAML` là STS API tương ứng.
- **If a backend service runs on EC2** → Dùng instance role, không dùng Cognito cho machine identity.

---

## 4. Bảo vệ API call bằng MFA với STS

### Question 4:
An IAM user must make programmatic AWS API calls that require MFA context. Which STS operation should the user call with an MFA code to obtain temporary credentials?

```
A. GetSessionToken
B. GetCallerIdentity
C. DecodeAuthorizationMessage
D. GetFederationToken
```

### ✅ Answer: **A. GetSessionToken**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- `GetSessionToken` nhận MFA device/code và trả access key ID, secret access key, session token cùng expiration.
- Policy có thể kiểm tra `aws:MultiFactorAuthPresent` cho API action cần MFA.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | `GetCallerIdentity` chỉ trả danh tính caller, không cấp MFA session. |
| C | `DecodeAuthorizationMessage` giải mã authorization failure message. |
| D | `GetFederationToken` tạo credentials cho federated user, không phải luồng MFA được hỏi. |

### 📝 Vocabulary Analysis:
- **MFA context**: thông tin cho biết request được xác thực bằng MFA.
- **expiration**: thời điểm credentials hết hạn.
- **programmatic API call**: gọi AWS API bằng SDK/CLI thay vì Console.

### 🔄 Context Variations:
- **If the user needs a different role's permissions** → Dùng `AssumeRole`; có thể yêu cầu MFA trong trust policy.
- **If the caller is the root user** → Không dùng root cho tác vụ hằng ngày; `GetSessionToken` không làm root session thành least-privilege.

---

## 5. Đọc kết quả IAM policy evaluation

### Question 5:
An IAM role has an identity policy that allows `s3:GetObject`, but an applicable policy explicitly denies the same action on the requested object. What is the authorization result?

```
A. Allow, because an Allow always overrides a Deny
B. Deny, because an explicit Deny overrides an Allow
C. Allow only when the object is smaller than 4 KB
D. Retry the request until the Deny expires
```

### ✅ Answer: **B. Deny, because an explicit Deny overrides an Allow**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Explicit Deny có precedence cao hơn Allow. Nếu không có explicit Deny, request vẫn cần có Allow; không có Allow thì mặc định là implicit deny.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| A | Trái với quy tắc IAM: explicit Deny luôn thắng. |
| C | Kích thước object không thay đổi kết quả policy evaluation. |
| D | Retry không gỡ bỏ policy Deny. |

### 📝 Vocabulary Analysis:
- **explicit Deny**: policy ghi rõ `Effect: Deny`.
- **implicit deny**: không có policy áp dụng cho phép request.
- **applicable policy**: policy có principal/action/resource/condition khớp request.

### 🔄 Context Variations:
- **If there is no Deny and an applicable Allow exists** → Request có thể được phép, nếu không bị policy giới hạn khác chặn.
- **If the request is cross-account** → Cần policy evaluation cho cả account của principal và account sở hữu resource.

---

## 6. Chia sẻ S3 bucket cross-account

### Question 6:
A role in Account A needs to read objects from a bucket in Account B. Which statement is correct for cross-account access?

```
A. Only the bucket policy in Account B must allow the request
B. Only the role's identity policy in Account A must allow the request
C. The principal-side identity policy and the resource-side bucket policy must both allow the request
D. The bucket must be made public
```

### ✅ Answer: **C. The principal-side identity policy and the resource-side bucket policy must both allow the request**

### 📖 Explanation (Vietnamese):

**Tại sao chọn C:**
- Với cross-account resource access, account của principal phải cho phép identity thực hiện action; resource policy ở account chứa bucket cũng phải cho principal truy cập.
- Explicit Deny hoặc policy giới hạn khác vẫn có thể từ chối request.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| A | Bucket policy đơn lẻ không thay thế permission phía identity/account nguồn trong cross-account access. |
| B | Identity policy đơn lẻ không cấp quyền vào resource thuộc account khác. |
| D | Public access không phải giải pháp chia sẻ cross-account an toàn. |

### 📝 Vocabulary Analysis:
- **principal-side**: phía account chứa IAM user/role gọi request.
- **resource-side**: phía account sở hữu resource cần truy cập.
- **bucket policy**: resource-based policy gắn trực tiếp với S3 bucket.

### 🔄 Context Variations:
- **If role and bucket are in the same account** → Resource policy và identity policy được đánh giá theo mô hình same-account; vẫn phải tính explicit Deny/boundary/SCP.
- **If the object is SSE-KMS encrypted** → Cần quyền sử dụng KMS key ngoài quyền S3.

---

## 7. Giới hạn user vào S3 prefix riêng

### Question 7:
A company needs to give each IAM user access only to `/home/<username>/` in one S3 bucket without creating a separate policy for every user. Which feature should the policy use?

```
A. The `${aws:username}` policy variable
B. A public bucket ACL
C. A separate root access key for each user
D. An S3 lifecycle rule
```

### ✅ Answer: **A. The `${aws:username}` policy variable**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Policy variable `${aws:username}` tạo resource path theo username của IAM user, giúp một policy áp dụng cho nhiều user với prefix riêng.
- Cách này mở rộng tốt hơn việc tạo một policy tĩnh cho từng người.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | ACL public mở quyền, không giới hạn user vào prefix riêng. |
| C | Root key không phải user identity và không được chia sẻ. |
| D | Lifecycle rule quản lý vòng đời object, không cấp quyền đọc/ghi. |

### 📝 Vocabulary Analysis:
- **policy variable**: biến policy được thay bằng context của request.
- **prefix**: phần đầu của S3 object key dùng để nhóm object theo đường dẫn logic.
- **static policy**: policy hard-code resource/user cụ thể.

### 🔄 Context Variations:
- **If each caller assumes an IAM role instead of using an IAM user** → Kiểm tra `aws:username` có tồn tại trong context hay không; có thể dùng principal/session tag.
- **If access should be read-only** → Giới hạn Action ở các thao tác đọc cần thiết thay vì cấp write.

---

## 8. Chọn loại IAM policy để quản lý tập trung

### Question 8:
A security team needs a reusable policy attached to several roles, with centralized versioning and rollback. Which policy type best matches this requirement?

```
A. Customer managed policy
B. Inline policy
C. AWS account root policy
D. S3 bucket ACL
```

### ✅ Answer: **A. Customer managed policy**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Customer Managed Policy có thể tái sử dụng cho nhiều principal, được quản lý version và rollback tập trung theo slide.
- Chính sách thay đổi có thể review ở một nơi thay vì sửa từng identity.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Inline Policy gắn một-một với principal và bị xóa cùng principal. |
| C | AWS không có “root policy” dùng làm permission template theo yêu cầu này. |
| D | ACL của S3 không phải IAM policy dùng chung cho nhiều role. |

### 📝 Vocabulary Analysis:
- **reusable**: có thể gắn cho nhiều principal.
- **centralized versioning**: quản lý các phiên bản policy tại một nơi.
- **inline policy**: policy nhúng trực tiếp vào một IAM identity.

### 🔄 Context Variations:
- **If AWS maintains the policy and the permissions are generic** → Có thể cân nhắc AWS Managed Policy, nhưng theo dõi thay đổi nội dung.
- **If the policy is intentionally inseparable from one principal** → Inline Policy có thể phù hợp.

---

## 9. Chọn AWS Directory Service

### Question 9:
A company must continue using its existing on-premises Microsoft Active Directory as the source of users. AWS applications need to authenticate through a directory proxy without replicating user accounts to a new AWS directory. Which service should it choose?

```
A. AD Connector
B. Simple AD
C. AWS Managed Microsoft AD with new local users only
D. Amazon Cognito User Pools
```

### ✅ Answer: **A. AD Connector**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- AD Connector làm directory gateway/proxy tới AD on-premises; user vẫn được quản lý tại directory hiện hữu.
- Phù hợp khi tổ chức muốn giữ AD làm nguồn danh tính.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Simple AD là directory tương thích AD, không phải proxy tới AD nguồn. |
| C | Managed Microsoft AD tạo directory do AWS quản lý; không khớp yêu cầu chỉ proxy tới directory sẵn có. |
| D | Cognito User Pools phục vụ danh tính user ứng dụng, không phải proxy AD cho AWS applications. |

### 📝 Vocabulary Analysis:
- **source of users**: directory chính lưu và quản lý user account.
- **directory proxy**: chuyển yêu cầu xác thực tới directory nguồn.
- **replicating**: sao chép dữ liệu danh tính sang một directory khác.

### 🔄 Context Variations:
- **If the application requires a full Microsoft AD directory hosted on AWS** → Xem xét AWS Managed Microsoft AD.
- **If adopting Simple AD for a new account** → Kiểm tra availability hiện hành; AWS không onboarding khách hàng mới theo tài liệu hiện tại.

---

## 10. Chọn encryption in flight

### Question 10:
A browser sends sensitive data to a website. The requirement is to encrypt the data while it travels over the network and decrypt it at the receiving endpoint. Which mechanism matches the requirement?

```
A. TLS/HTTPS encryption in transit
B. S3 Lifecycle expiration
C. Server-side encryption at rest only
D. An IAM permission boundary
```

### ✅ Answer: **A. TLS/HTTPS encryption in transit**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- TLS bảo vệ data trên đường truyền; endpoint giải mã request sau khi nhận.
- Certificate TLS hỗ trợ HTTPS và giảm nguy cơ bị đọc/sửa dữ liệu giữa client và server.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Lifecycle expiration xóa object theo rule, không mã hóa network traffic. |
| C | At-rest encryption bảo vệ dữ liệu lưu trữ, không tự bảo vệ request trên mạng. |
| D | Permission boundary giới hạn quyền IAM, không phải cơ chế encryption. |

### 📝 Vocabulary Analysis:
- **in transit**: dữ liệu đang truyền qua network.
- **endpoint**: điểm kết thúc TLS như browser, load balancer hoặc API endpoint.
- **man-in-the-middle (MITM)**: kẻ xen giữa để đọc hoặc sửa traffic.

### 🔄 Context Variations:
- **If the requirement is to protect stored object data** → Dùng server-side encryption phù hợp, có thể quản lý key bằng KMS.
- **If the server must never read the payload** → Mã hóa client-side trước khi gửi.

---

## 11. Chọn client-side encryption

### Question 11:
A data owner requires that the storage service must **never be able to decrypt** uploaded documents. Which design best meets this requirement?

```
A. Encrypt the documents on the client and let only authorized clients decrypt them
B. Enable server-side encryption and give the service unrestricted key access
C. Use an S3 lifecycle rule
D. Enable a public bucket policy
```

### ✅ Answer: **A. Encrypt the documents on the client and let only authorized clients decrypt them**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Client-side encryption giữ plaintext/key ở client được ủy quyền; server chỉ nhận ciphertext.
- Khác server-side encryption, nơi service có thể giải mã khi xử lý request được cấp quyền.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Nếu service có quyền dùng key để giải mã, không đảm bảo service không thể đọc plaintext. |
| C | Lifecycle rule không mã hóa dữ liệu. |
| D | Public access làm tăng nguy cơ lộ ciphertext và không đáp ứng key control. |

### 📝 Vocabulary Analysis:
- **client-side encryption**: client mã hóa trước khi gửi dữ liệu tới server.
- **ciphertext**: dữ liệu đã mã hóa.
- **authorized client**: client có quyền và key cần thiết để giải mã.

### 🔄 Context Variations:
- **If AWS service must process plaintext but storage must be encrypted** → Server-side encryption có thể phù hợp.
- **If encrypting a large file client-side** → Dùng Envelope Encryption để xử lý data key.

---

## 12. Mã hóa payload lớn bằng KMS

### Question 12:
An application must encrypt a 10 MB file using AWS KMS. Which approach should it use according to the course material?

```
A. Use Envelope Encryption with the GenerateDataKey API
B. Send the entire file to the KMS Encrypt API
C. Store the file in Parameter Store Standard tier
D. Convert the file to an IAM policy
```

### ✅ Answer: **A. Use Envelope Encryption with the GenerateDataKey API**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Slide nêu KMS Encrypt API có giới hạn 4 KB; file lớn cần Envelope Encryption.
- `GenerateDataKey` trả plaintext data key để client mã hóa file và encrypted data key để lưu cùng ciphertext.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | KMS Encrypt không nhận trực tiếp payload lớn như 10 MB. |
| C | Parameter Store dùng cho parameter/configuration, không phải storage file. |
| D | IAM policy là tài liệu quyền, không phải encryption format. |

### 📝 Vocabulary Analysis:
- **Envelope Encryption**: mã hóa data bằng data key rồi mã hóa data key bằng KMS Key.
- **GenerateDataKey**: KMS API tạo plaintext và encrypted version của data key.
- **encrypted data key**: data key đã được KMS Key mã hóa để lưu cùng ciphertext.

### 🔄 Context Variations:
- **If the plaintext is under the KMS Encrypt API limit** → Có thể dùng Encrypt/Decrypt API trực tiếp.
- **If using a standardized client-side envelope format** → Cân nhắc AWS Encryption SDK.

---

## 13. Cấp quyền cross-account cho KMS Key

### Question 13:
A principal in Account B must decrypt data using a customer managed KMS key owned by Account A. Which setup is required?

```
A. Allow the principal/account in the key policy in Account A and grant the principal IAM permission in Account B
B. Add an S3 bucket ACL in Account B only
C. Make the KMS key public
D. Change the KMS key to an AWS owned key
```

### ✅ Answer: **A. Allow the principal/account in the key policy in Account A and grant the principal IAM permission in Account B**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- KMS Key Policy tại account sở hữu key phải cho phép cross-account principal/account.
- IAM policy ở Account B cũng cần cho phép principal gọi `kms:Decrypt` trên key ARN.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | S3 ACL không cấp quyền sử dụng KMS Key. |
| C | KMS Key không nên được mở public; key policy kiểm soát principal cụ thể. |
| D | AWS owned key không phải key do customer tùy ý share cross-account theo cách nêu. |

### 📝 Vocabulary Analysis:
- **customer managed KMS key**: KMS Key khách hàng kiểm soát policy/configuration.
- **key policy**: resource policy bắt buộc để điều khiển quyền KMS Key.
- **kms:Decrypt**: quyền yêu cầu KMS giải mã ciphertext/data key.

### 🔄 Context Variations:
- **If the role and key are in the same account** → Kiểm tra key policy có cho IAM policy delegation hay cấp trực tiếp quyền.
- **If the data is an encrypted EBS snapshot** → Share cả snapshot và key access, rồi copy/re-encrypt trong account đích.

---

## 14. Copy EBS snapshot encrypted sang Region khác

### Question 14:
An EBS snapshot encrypted with KMS Key A in `eu-west-2` must be copied to `ap-southeast-2`. Which action should be taken in the destination Region?

```
A. Copy the snapshot and encrypt the copy with a KMS key available in the destination Region
B. Reuse the source Region's KMS key ARN without copying the snapshot
C. Disable encryption on the source volume
D. Make KMS Key A public
```

### ✅ Answer: **A. Copy the snapshot and encrypt the copy with a KMS key available in the destination Region**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Snapshot copy sang Region đích cần được mã hóa lại bằng key dùng được tại Region đích.
- Slide minh họa KMS re-encrypt bằng Key B khi chuyển từ key/Region nguồn sang đích.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | KMS Key là regional resource; key ARN ở Region nguồn không thay thế key đích. |
| C | Tắt encryption không phải yêu cầu để copy snapshot an toàn. |
| D | Không bao giờ mở key public để giải quyết vấn đề cross-Region. |

### 📝 Vocabulary Analysis:
- **snapshot copy**: bản sao EBS snapshot được tạo ở Region khác.
- **re-encrypt**: mã hóa lại ciphertext bằng KMS Key đích.
- **destination Region**: Region chứa snapshot copy cần sử dụng.

### 🔄 Context Variations:
- **If the snapshot is shared cross-account** → Cấp quyền snapshot và KMS key, sau đó account đích copy bằng key của mình.
- **If the snapshot is unencrypted** → Encryption có thể được chọn trong quá trình copy theo yêu cầu bảo mật.

---

## 15. Chọn CloudHSM

### Question 15:
A security requirement calls for dedicated HSM hardware and customer control of encryption keys rather than AWS-managed KMS key material. Which service best matches the requirement?

```
A. AWS CloudHSM
B. AWS Secrets Manager
C. Amazon Athena
D. Amazon SES
```

### ✅ Answer: **A. AWS CloudHSM**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- CloudHSM cung cấp HSM chuyên dụng; khách hàng tự quản lý encryption key và dùng CloudHSM Client Software.
- Khác KMS managed key workflow, CloudHSM yêu cầu thêm trách nhiệm về client, cluster và vận hành.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Secrets Manager lưu secret và hỗ trợ rotation, không cung cấp dedicated HSM cho khách hàng quản lý key. |
| C | Athena là dịch vụ SQL analytics trên S3. |
| D | SES là dịch vụ email. |

### 📝 Vocabulary Analysis:
- **dedicated HSM**: phần cứng Hardware Security Module chuyên dụng.
- **key material**: bí mật mật mã thực sự được dùng trong operation.
- **CloudHSM Client Software**: phần mềm client kết nối tới CloudHSM cluster.

### 🔄 Context Variations:
- **If the application needs managed key operations integrated with AWS services** → KMS thường đơn giản hơn.
- **If the requirement is only to encrypt an application secret** → Secrets Manager hoặc Parameter Store có thể phù hợp hơn.

---

## 16. Chọn Parameter Store tier

### Question 16:
An application must store a 6 KB configuration value and use parameter policies. Which Parameter Store tier supports both requirements according to the course material?

```
A. Advanced
B. Standard
C. AWS Managed KMS key
D. S3 Glacier Flexible Retrieval
```

### ✅ Answer: **A. Advanced**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Theo bảng slide, Advanced hỗ trợ value tối đa 8 KB và Parameter Policies; Standard giới hạn 4 KB và không hỗ trợ Parameter Policies.
- Advanced có chi phí lưu trữ theo slide, nên cần kiểm tra quota/pricing hiện hành.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Standard không đáp ứng kích thước 6 KB và parameter policy theo bảng slide. |
| C | KMS key là encryption key, không phải Parameter Store tier. |
| D | S3 Glacier là storage class, không cung cấp Parameter Store parameter policy. |

### 📝 Vocabulary Analysis:
- **parameter tier**: cấp Standard/Advanced với giới hạn và tính năng khác nhau.
- **parameter policy**: policy áp dụng cho Advanced parameter như expiration/notification use case.
- **configuration value**: giá trị cấu hình mà ứng dụng đọc khi chạy.

### 🔄 Context Variations:
- **If the value is under 4 KB and no parameter policy is needed** → Standard có thể phù hợp và rẻ hơn.
- **If the value is a database password requiring managed rotation** → Cân nhắc Secrets Manager.

---

## 17. Tự động rotation database secret

### Question 17:
An application uses an Amazon RDS database password. The team wants AWS to rotate the secret periodically using a managed Lambda rotation workflow. Which service should it choose?

```
A. AWS Secrets Manager
B. SSM Parameter Store Standard without custom automation
C. AWS Certificate Manager
D. Amazon Macie
```

### ✅ Answer: **A. AWS Secrets Manager**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Secrets Manager được thiết kế cho secret lifecycle và hỗ trợ rotation bằng Lambda; có rotation integration/template cho một số database như RDS.
- Secret được mã hóa bằng KMS và có thể lấy qua API theo IAM permission.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Parameter Store không có built-in rotation theo slide; cần tự xây Lambda/EventBridge workflow. |
| C | ACM quản lý TLS certificate, không quản lý database password. |
| D | Macie tìm sensitive data, không rotate credential. |

### 📝 Vocabulary Analysis:
- **secret rotation**: thay credential theo lịch và cập nhật consumer/database.
- **rotation workflow**: chuỗi bước tạo secret mới, đổi database và đồng bộ secret store.
- **managed template**: cấu hình rotation được AWS cung cấp cho loại resource được hỗ trợ.

### 🔄 Context Variations:
- **If only a non-secret configuration value needs versioning** → Parameter Store phù hợp hơn.
- **If using a database without a provided rotation template** → Thiết kế/test custom Lambda rotation trước khi bật production schedule.

---

## 18. Replicate secret đa Region

### Question 18:
A multi-Region application needs read access to the same secret in a secondary Region, with the replica kept synchronized with the primary secret. Which feature should it use?

```
A. Secrets Manager multi-Region secret replication
B. A public S3 object containing the password
C. An IAM inline policy
D. An Athena federated connector
```

### ✅ Answer: **A. Secrets Manager multi-Region secret replication**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Secrets Manager có thể tạo read replica ở Region khác và đồng bộ với primary secret; slide nêu có thể promote replica thành standalone secret.
- Phù hợp ứng dụng đa Region hoặc disaster recovery.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Public S3 object làm lộ password và không duy trì lifecycle an toàn. |
| C | Inline policy cấp quyền, không replicate secret data. |
| D | Athena connector query data source, không đồng bộ secret. |

### 📝 Vocabulary Analysis:
- **read replica**: bản sao dùng để đọc, đồng bộ từ primary.
- **primary secret**: secret nguồn quản lý phiên bản chính.
- **promote**: chuyển replica thành secret độc lập theo thao tác được hỗ trợ.

### 🔄 Context Variations:
- **If only one Region consumes the secret** → Không cần tạo replica đa Region.
- **If a regional disaster requires failover** → Kiểm thử promote/failover và cập nhật ứng dụng trước sự cố.

---

## 19. Gửi email transactional bằng SES

### Question 19:
An application must send transactional and marketing email at scale and monitor delivery, bounce, and sender reputation metrics. Which AWS service is designed for this purpose?

```
A. Amazon SES
B. Amazon Athena
C. Amazon OpenSearch Service
D. AWS Directory Service
```

### ✅ Answer: **A. Amazon SES**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- SES là dịch vụ email managed, hỗ trợ gửi/nhận email qua API hoặc SMTP và cung cấp delivery/bounce/reputation metrics.
- DKIM/SPF hỗ trợ xác thực domain gửi.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Athena chạy SQL analytics trên data lưu ở S3. |
| C | OpenSearch phục vụ search/analytics trên index. |
| D | Directory Service quản lý/kết nối directory, không gửi email. |

### 📝 Vocabulary Analysis:
- **transactional email**: email phục vụ sự kiện ứng dụng như receipt/reset password.
- **bounce**: email không chuyển phát được.
- **sender reputation**: đánh giá uy tín nguồn gửi email.

### 🔄 Context Variations:
- **If the app only needs to query email delivery logs in S3** → Dùng Athena để phân tích log, không thay SES làm mail sender.
- **If the requirement is to authenticate the sender domain** → Xác minh SES identity và cấu hình DKIM/SPF.

---

## 20. Tìm kiếm partial match trên nhiều field

### Question 20:
A DynamoDB application needs full-text search and partial matches across fields that are not the table's primary key. Which service can complement DynamoDB for this search requirement?

```
A. Amazon OpenSearch Service
B. AWS Certificate Manager
C. AWS STS
D. Amazon SES
```

### ✅ Answer: **A. Amazon OpenSearch Service**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- OpenSearch tạo search index hỗ trợ tìm theo field/partial match; slide mô tả dịch vụ này bổ sung cho database như DynamoDB.
- DynamoDB giữ vai trò transactional/source data; OpenSearch phục vụ search workload.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | ACM quản lý TLS certificate. |
| C | STS phát hành temporary credentials. |
| D | SES gửi email. |

### 📝 Vocabulary Analysis:
- **full-text search**: tìm nội dung văn bản thay vì chỉ tìm theo khóa chính xác.
- **partial match**: khớp một phần nội dung/từ khóa.
- **complement a database**: bổ sung query capability mà database chính không tối ưu.

### 🔄 Context Variations:
- **If the requirement is lookup by DynamoDB primary key** → Query DynamoDB trực tiếp.
- **If using SQL against an OpenSearch domain** → Xác nhận plugin/engine version và SQL support hiện hành; tài liệu mới có SQL support cho các version phù hợp.

---

## 21. Chạy serverless SQL trên S3

### Question 21:
An analyst needs to run ad hoc SQL queries directly on CSV and Parquet files in Amazon S3 without managing database servers. Which service should the analyst use?

```
A. Amazon Athena
B. AWS CloudHSM
C. AWS Directory Service
D. AWS Secrets Manager
```

### ✅ Answer: **A. Amazon Athena**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Athena là serverless SQL query service trên file trong S3; slide liệt kê CSV, JSON, ORC, Avro và Parquet.
- Không cần tự quản lý query server; chi phí theo lượng data scan theo mô tả khóa học.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | CloudHSM là HSM dùng cho key operations. |
| C | Directory Service quản lý/kết nối AD. |
| D | Secrets Manager lưu và rotate secret. |

### 📝 Vocabulary Analysis:
- **ad hoc query**: truy vấn phát sinh theo nhu cầu phân tích.
- **serverless SQL**: chạy query mà không quản lý server database.
- **data scanned**: lượng dữ liệu Athena đọc để trả lời query.

### 🔄 Context Variations:
- **If the same data is queried frequently with low latency needs** → Đánh giá database/warehouse phù hợp thay vì scan object mỗi lần.
- **If query cost is high** → Dùng columnar format, compression, partition và chỉ chọn cột cần thiết.

---

## 22. Query nhiều data source bằng Athena Federated Query

### Question 22:
A team needs SQL queries over data in Amazon RDS and DynamoDB without first copying all data to S3. Which Athena capability uses data source connectors for this purpose?

```
A. Athena Federated Query
B. Amazon SES inbound email processing
C. AWS Certificate Manager renewal
D. KMS automatic key rotation
```

### ✅ Answer: **A. Athena Federated Query**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Federated Query dùng Data Source Connector, chạy trên Lambda, để query data source ngoài S3 như RDS hoặc DynamoDB.
- Kết quả query có thể được ghi về S3.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | SES xử lý email, không tạo SQL connector. |
| C | ACM quản lý TLS certificate và renewal. |
| D | KMS rotation quản lý key material, không query database. |

### 📝 Vocabulary Analysis:
- **federated query**: query qua connector tới nhiều loại data source.
- **data source connector**: thành phần kết nối Athena với nguồn dữ liệu cụ thể.
- **query result**: dữ liệu được trả về sau khi thực thi SQL.

### 🔄 Context Variations:
- **If all data is already in S3** → Dùng Athena query trực tiếp, không cần connector federated.
- **If the connector must access private RDS** → Cấu hình VPC/network/IAM và kiểm tra Lambda connector requirements.

---

## 23. Cấp và renew TLS certificate

### Question 23:
A website behind an Elastic Load Balancer needs a managed TLS certificate for HTTPS. The team also wants certificate renewal handled automatically when renewal conditions are met. Which service should it use?

```
A. AWS Certificate Manager (ACM)
B. Amazon Macie
C. Amazon Athena
D. AWS STS
```

### ✅ Answer: **A. AWS Certificate Manager (ACM)**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- ACM cấp/quản lý TLS certificate và tích hợp với Elastic Load Balancer; ACM hỗ trợ managed renewal khi validation và integration thỏa điều kiện.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | Macie khám phá dữ liệu nhạy cảm, không cấp TLS certificate. |
| C | Athena chạy SQL query trên S3. |
| D | STS cấp temporary credentials. |

### 📝 Vocabulary Analysis:
- **TLS certificate**: chứng chỉ dùng thiết lập kết nối HTTPS/TLS.
- **managed renewal**: dịch vụ tự thực hiện gia hạn khi đáp ứng điều kiện.
- **validation**: bước xác minh quyền kiểm soát domain/certificate.

### 🔄 Context Variations:
- **If the endpoint is CloudFront or API Gateway** → ACM cũng tích hợp theo phạm vi/Region yêu cầu.
- **If using a private certificate authority** → Kiểm tra ACM Private CA và pricing; public certificate miễn phí không đồng nghĩa private CA miễn phí.

---

## 24. Phát hiện PII trong AWS data

### Question 24:
A security team wants a managed service that uses machine learning and pattern matching to discover sensitive data such as personally identifiable information in AWS. Which service should it use?

```
A. Amazon Macie
B. Amazon SES
C. Amazon OpenSearch Service
D. AWS CodeBuild
```

### ✅ Answer: **A. Amazon Macie**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- Macie là dịch vụ data security/privacy dùng Machine Learning và pattern matching để phát hiện dữ liệu nhạy cảm như PII.
- Findings giúp đội security xác định vị trí cần điều tra/khắc phục.

**Tại sao loại các đáp án khác:**

| Option | Lý do loại bỏ |
|---|---|
| B | SES gửi/nhận email và theo dõi delivery metrics. |
| C | OpenSearch tạo search/analytics index, không phải dịch vụ phát hiện PII chuyên biệt theo yêu cầu. |
| D | CodeBuild build/test source code. |

### 📝 Vocabulary Analysis:
- **personally identifiable information (PII)**: dữ liệu có thể nhận dạng cá nhân.
- **pattern matching**: dò dữ liệu khớp mẫu nhận dạng đã biết.
- **finding**: kết quả cảnh báo/phát hiện cần review.

### 🔄 Context Variations:
- **If the requirement is to block unauthorized reads** → Dùng IAM/resource policy và encryption; Macie discovery không thay authorization.
- **If a finding identifies a sensitive object** → Xác nhận owner/context rồi xử lý permission, exposure và retention.
