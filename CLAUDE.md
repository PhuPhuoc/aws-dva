# AWS DVA Study Guide - Project Documentation

## Project Purpose

This is a detailed study documentation set for the **AWS Certified Developer - Associate (DVA-C02)** exam, generated from Cloud Mentor Pro's PDF material.

**Source file:** `[CMP-DVA] AWS Certified Developer - Associate_v5.pdf`

**Output:** `.md` files within `Section_N/` directories

---

## Directory Structure

```
aws-dva-ai-gen/
├── [CMP-DVA] AWS Certified Developer - Associate_v5.pdf
├── CLAUDE.md
├── Section1/
│   ├── slide.md          # Translated slide content (preserving original structure)
│   ├── explain_more.md   # Extended knowledge based on practical experience
│   └── dva_example.md    # DVA practice questions + answers + explanations
├── Section2/
│   └── ...
└── Section_N/
    └── ...

```

> **NOTE:** No need to create an `images/` folder or `context.txt` file. Users will view slides directly from the PDF when needed.

---

## Workflow for Creating a New Section

### Step 1: Extract text from PDF

```bash
# Extract text from page X to page Y
pdftotext -f [start] -l [end] "[path/to/file.pdf]" -

```

### Step 2: Create 3 files for the Section

| File | Purpose |
| --- | --- |
| `slide.md` | Slide content translated → Vietnamese |
| `explain_more.md` | Extended knowledge based on practical experience |
| `dva_example.md` | DVA practice questions + detailed answers |

### Step 3: Verification and commit

---

## Translation Rules (STRICT COMPLIANCE REQUIRED)

### Core Principle

> **Core Rule:** Content must strictly align with the original slides. All extended content must be clearly tagged as **DO (NÊN)** or **DON'T (KHÔNG NÊN)** for easy distinction.

### List of Keywords NOT to Translate

Keep in English:

| No. | Keyword | Notes |
| --- | --- | --- |
| 1 | AWS | - |
| 2 | IAM | Identity and Access Management |
| 3 | EC2 | Elastic Compute Cloud |
| 4 | S3 | Simple Storage Service |
| 5 | Region | - |
| 6 | Availability Zone (AZ) | - |
| 7 | Security Group | - |
| 8 | IAM Role | - |
| 9 | IAM Policy | - |
| 10 | MFA | Multi-Factor Authentication |
| 11 | CLI | Command Line Interface |
| 12 | SDK | Software Development Kit |
| 13 | EBS | Elastic Block Store |
| 14 | VPC | Virtual Private Cloud |
| 15 | Lambda | - |
| 16 | RDS | Relational Database Service |
| 17 | Auto Scaling | - |
| 18 | Load Balancer | - |
| 19 | CloudWatch | - |
| 20 | CloudTrail | - |
| 21 | SSM | Systems Manager |
| 22 | SQS | Simple Queue Service |
| 23 | SNS | Simple Notification Service |
| 24 | API Gateway | - |
| 25 | Docker / Container | - |
| 26 | Kubernetes / EKS | - |
| 27 | ECS | Elastic Container Service |
| 28 | Fargate | - |
| 29 | CodeBuild | - |
| 30 | CodeDeploy | - |
| 31 | CodePipeline | - |
| 32 | X-Ray | - |
| 33 | DynamoDB | - |
| 34 | ElastiCache | - |
| 35 | API | Application Programming Interface |
| 36 | JSON | JavaScript Object Notation |
| 37 | REST | Representational State Transfer |
| 38 | HTTP | HyperText Transfer Protocol |
| 39 | HTTPS | HTTP Secure |
| 40 | SSL / TLS | Secure Sockets Layer / Transport Layer Security |
| 41 | SSH | Secure Shell |
| 42 | VPC Endpoint | - |
| 43 | NAT Gateway | - |
| 44 | Bastion Host | - |
| 45 | VPN | Virtual Private Network |
| 46 | Direct Connect | - |
| 47 | WAF | Web Application Firewall |
| 48 | Shield | - |
| 49 | Cognito | - |
| 50 | Secrets Manager | - |
| 51 | Parameter Store | - |
| 52 | CloudFormation | - |
| 53 | SAM | Serverless Application Model |
| 54 | SAM CLI | SAM Command Line Interface |
| 55 | CDK | Cloud Development Kit |
| 56 | Elastic Beanstalk | - |
| 57 | On-Demand | - |
| 58 | Reserved Instance | - |
| 59 | Spot Instance | - |
| 60 | Savings Plans | - |
| 61 | Dedicated Host | - |
| 62 | Auto Scaling Group (ASG) | - |
| 63 | Launch Template | - |
| 64 | Launch Configuration | - |
| 65 | AMI | Amazon Machine Image |
| 66 | Instance Store | - |
| 67 | Instance Type | - |
| 68 | Key Pair | - |
| 69 | User Data | - |
| 70 | Systems Manager Session Manager | - |
| 71 | State Manager | - |
| 72 | Run Command | - |

---

## Detailed Structure for Each File

### 1. slide.md

**Rules:**

* Translate slide text directly into Vietnamese
* Preserve bullet point structures and tables
* **Keep technical keywords in English**
* **Main titles must reference slide page numbers** (e.g., `## AWS Regions (Pages 10-11)`)
* Diagram descriptions (if analyzed) should be *italicized* or placed in a blockquote `> *...*`

**Example:**

```markdown
## AWS Regions (Pages 10-11)

- **Region** is a cluster of data centers.

> *Diagram: Illustrating AWS global regions*

```

### 2. explain_more.md

**Mục đích:** Bổ sung kiến thức thực tế từ kinh nghiệm triển khai AWS mà slide không đề cập.

**Cấu trúc bắt buộc:**

Mỗi topic phải có đủ 3 phần:

```markdown
### [Topic Name]

#### Giải thích cốt lõi (bắt buộc)
[Bắt buộc phải có: Giải thích CƠ CHẾ hoạt động, TẠI SAO nó như vậy, SO SÁNH với alternatives]

#### NÊN làm (DO)
- **Thực hành tốt #1** → Giải thích ngắn gọn TẠI SAO nên làm, và NÊN làm SO VỚI CÁI GÌ
- **Thực hành tốt #2** → Tương tự

#### KHÔNG NÊN làm (DON'T)
- **Anti-pattern #1** → Giải thích HẬU QUẢ/KẾT QUẢ nếu làm, và NÊN làm gì thay thế
- **Anti-pattern #2** → Tương tự
```

**Quy tắc viết:**

| Phần | Yêu cầu |
|------|----------|
| **Giải thích cốt lõi** | BẮT BUỘC - Giải thích how it works, tại sao, comparison với alternatives |
| **NÊN** | Phải nói rõ "nên làm X **vì** Y, **so với** Z" |
| **KHÔNG NÊN** | Phải nói rõ "không nên làm X **vì** sẽ bị Y, **nên làm** Z thay thế |

**Ví dụ format đúng:**

```markdown
### EBS Volume Size

#### Giải thích cốt lõi
EBS volume có thể **tăng size** bằng modify operation (không downtime), nhưng **KHÔNG thể giảm size** trực tiếp.

Để "giảm" EBS, phải:
1. Tạo snapshot backup (phòng data loss)
2. Tạo volume mới (size nhỏ hơn)
3. Attach vào instance
4. Mount, copy data, unmount volume cũ
→ Mất thời gian, cần downtime, phức tạp

**So sánh các options:**
| Cách | Upsize (thêm) | Downsize (bớt) |
|------|---------------|-----------------|
| EBS Volume trực tiếp | ✅ Online, không downtime | ❌ Phải manual swap |
| Instance Store | ✅ Local, rất nhanh | ❌ Không thể resize, ephemeral |
| EFS | ✅ Auto-scaling | ✅ Auto-scaling | 

#### NÊN làm:
- **Bắt đầu với size nhỏ vừa đủ, tăng dần** → Vì upsizing dễ (1-click), không downtime. Downsizing khó và tốn công.
- **Dùng gp3 thay vì gp2** → gp3 rẻ hơn, cho phép tăng IOPS và throughput độc lập với size.

#### KHÔNG NÊN làm:
- **Cấu hình EBS quá lớn từ đầu** → Vì nếu sau này cần giảm, phải làm thủ công phức tạp. NÊN: bắt đầu nhỏ, resize khi có data thật.
- **Dùng gp2 cho database mới** → Vì gp2 có IOPS linked với size (3 IOPS/GB). Nếu cần nhiều IOPS mà data nhỏ, phải overprovision size. NÊN: dùng gp3 với provisioned IOPS.
```

**Ví dụ format SAI (cần tránh):**

```markdown
### EBS Volume Size

**NÊN:**
- Bắt đầu nhỏ vì có thể tăng

**KHÔNG NÊN:**
- Không nên cấu hình lớn từ đầu

# ❌ SAI: Không giải thích TẠI SAO, không SO SÁNH, không nói HẬU QUẢ
```

**Checklist cho mỗi topic trong explain_more.md:**
- [ ] Có giải thích CƠ CHẾ hoạt động?
- [ ] Có giải thích TẠI SAO nên/không nên?
- [ ] Có SO SÁNH với alternatives?
- [ ] Có nói rõ HẬU QUẢ nếu làm sai?
- [ ] Có suggest action thay thế cho anti-patterns?

### 3. dva_example.md

**Rules (FROM SESSION 2 ONWARDS):**

* **Questions written in ENGLISH** (identical to the official exam format)
* **Explanations written in VIETNAMESE**
* **Vocabulary analysis** provided when useful (technical terms, collocations)
* **Highlight keywords** inside questions for quick recall

**Standard Format:**

```markdown
## [Topic] - [Sub-topic]

### Question:
[Question in English, highlight **keywords**]


```

[A, B, C, D options]

```

### ✅ Answer: [Correct Answer]

### 📖 Explanation (Vietnamese):

**Why choose [answer]:**
[Detailed explanation in Vietnamese]

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A | [Reason] |
| B | [Reason] |

### 📝 Vocabulary Analysis:
- **keyword1**: definition / meaning in context
- **keyword2**: definition / meaning in context

### 🔄 Context Variations:
- **If [change X]** → [a different answer becomes correct]
- **If [change Y]** → [what additional conditions are required]

```

---

## Helper Tools

### PDF Tools (poppler-utils installed)

```bash
# Extract text from PDF
pdftotext -f [start] -l [end] "[path/to/file.pdf]" -

# View PDF information/metadata
pdfinfo "[path/to/file.pdf]"

# Convert PDF pages to PNG images (if needed)
pdftoppm -png "[path/to/file.pdf]" output_prefix

```

---

## Checklist Before Finishing a Section

* [ ] `slide.md`: Accurate translation, preserved keywords, correct page numbers
* [ ] `explain_more.md`: Contains DO/DON'T tags, tips, and warnings
* [ ] `dva_example.md`: English questions, Vietnamese explanations, vocabulary analysis included
* [ ] `CLAUDE.md`: Updated if project changes occur

---

## How to Use This Documentation

### Recommended Learning Flow:

1. **Read `slide.md**` – Master foundational knowledge from original slides.
2. **Open PDF/Slides side-by-side** – Review illustrations and diagrams manually.
3. **Read `explain_more.md**` – Deepen understanding through real-world operational context.
4. **Practice with `dva_example.md**` – Evaluate knowledge retention with exam-style questions.

### Memory Tips:

* **DO (NÊN)** = Best practice = Frequently the correct answer in exams.
* **DON'T (KHÔNG NÊN)** = Anti-pattern = Frequently used as trap options.
* Highlight topics or concepts that are hard to retain.
* Build flashcards using the questions from `dva_example.md`.