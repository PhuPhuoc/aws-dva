# Section 6 - DVA Practice Questions: CloudFront Advanced, Elastic Beanstalk, CloudFormation, Messaging

---

## 1. CloudFront Advanced

### Question 1:
A company wants to distribute premium video content to paid subscribers only. What should be configured?

```
A. CloudFront Geo Restriction
B. CloudFront Signed URLs
C. CloudFront Origin Groups
D. CloudFront Price Classes
```

### ✅ Answer: **B. CloudFront Signed URLs**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Signed URLs** cho phép:
  - Distribute **paid shared content** to premium users
  - Attach policy với **URL expiration**
  - Include **IP ranges**
  - Define **trusted signers**

**Use cases:**
- Premium video streaming
- Private document downloads
- User-specific content

**Signed URL vs Signed Cookies:**
- **Signed URL**: Individual files
- **Signed Cookies**: Multiple files

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Geo Restriction | Country-based access, không phải paid content |
| C. Origin Groups | Failover giữa origins |
| D. Price Classes | Chỉ giảm chi phí, không access control |

---

### Question 2:
Content was updated on the origin, but CloudFront is still serving old cached content. What should be done?

```
A. Update TTL
B. Create CloudFront Invalidation
C. Delete CloudFront distribution
D. Update Security Groups
```

### ✅ Answer: **B. Create CloudFront Invalidation**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Invalidation** forces cache refresh
- Bypasses TTL
- Can invalidate specific paths or all files

**Commands:**
```bash
# Specific path
aws cloudfront create-invalidation \
  --distribution-id EDFDVBD6EXAMPLE \
  --paths "/images/*"

# All files
aws cloudfront create-invalidation \
  --distribution-id EDFDVBD6EXAMPLE \
  --paths "/*"
```

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Update TTL | TTL chỉ affects future caching |
| C. Delete distribution | Extreme, unnecessary |
| D. Security Groups | Không liên quan đến caching |

---

## 2. Elastic Beanstalk

### Question 3:
Which Elastic Beanstalk deployment mode provides zero downtime with the **quickest rollback capability**?

```
A. All at once
B. Rolling
C. Immutable
D. Blue/Green
```

### ✅ Answer: **C. Immutable**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **Immutable deployment**:
  - **Zero downtime**
  - New instances in **temporary ASG**
  - New code deployed to new instances
  - Swap instances when healthy
  - **Quickest rollback**: Just terminate new ASG
  - High cost (double capacity)

**Rollback comparison:**
| Mode | Rollback Speed |
|------|---------------|
| All at once | Manual redeploy |
| Rolling | Manual rollback |
| Immutable | **Instant** (terminate ASG) |
| Blue/Green | Quick (CNAME swap) |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. All at once | Có downtime, rollback manual |
| B. Rolling | Partial downtime, manual rollback |
| D. Blue/Green | Nhanh nhưng không nhanh bằng terminate ASG |

---

### Question 4:
A company wants to deploy a new version to a production environment with **zero downtime** and minimal additional cost. Which deployment option is best?

```
A. All at once
B. Rolling with additional batches
C. Immutable
D. Traffic Splitting
```

### ✅ Answer: **B. Rolling with additional batches**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Rolling with additional batches**:
  - **Zero downtime**
  - Maintains **full capacity** during deployment
  - Small additional cost (additional batch instances)
  - Good for **production**

**Comparison of options:**
| Option | Zero Downtime | Additional Cost |
|--------|--------------|-----------------|
| All at once | ❌ | None |
| Rolling | ⚠️ | None |
| **Rolling+Batch** | ✅ | Small |
| Immutable | ✅ | High |
| Blue/Green | ✅ | Medium |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. All at once | Có downtime |
| C. Immutable | High cost (double capacity) |
| D. Traffic Splitting | Có thể automated rollback, nhưng rolling+batch minimal cost |

---

### Question 5:
An application is deployed on Elastic Beanstalk with RDS. For production, what is the **best practice** for database configuration?

```
A. Provision RDS within Beanstalk
B. Create RDS separately and provide connection string
C. Use DynamoDB instead
D. Remove database entirely
```

### ✅ Answer: **B. Create RDS separately and provide connection string**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Production best practice**:
  - Database lifecycle **separate** from Beanstalk
  - Create RDS **independently**
  - Provide connection string via environment variables
  - RDS won't be deleted when environment is terminated

**RDS with Beanstalk:**
| Environment | Recommendation |
|------------|----------------|
| Dev/Test | Provision with Beanstalk ✓ |
| **Production** | **Separate RDS** ✓ |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Provision with Beanstalk | Ties database lifecycle to environment |
| C. Use DynamoDB | Changes architecture, not always suitable |
| D. Remove database | May be required for the application |

---

### Question 6:
What file format is used for Elastic Beanstalk configuration extensions?

```
A. JSON only
B. YAML only
C. YAML or JSON
D. XML only
```

### ✅ Answer: **C. YAML or JSON**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **Elastic Beanstalk Extensions**:
  - `.ebextensions/` directory
  - YAML or JSON format
  - `.config` extension
  - `option_settings` for modifying defaults

**Example:**
```yaml
# .ebextensions/options.config
option_settings:
  aws:autoscaling:launchconfiguration:
    InstanceType: t3.medium
```

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. JSON only | YAML also supported |
| B. YAML only | JSON also supported |
| D. XML | Not supported |

---

## 3. CloudFormation

### Question 7:
Which CloudFormation **intrinsic function** is used to reference parameters and return resource IDs?

```
A. Fn::GetAtt
B. Fn::Ref
C. Fn::Sub
D. Fn::ImportValue
```

### ✅ Answer: **B. Fn::Ref**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Ref function**:
  - Reference **parameters** → returns value
  - Reference **resources** → returns physical ID
  - YAML shorthand: `!Ref`

**Examples:**
```yaml
# Reference parameter
InstanceType: !Ref InstanceTypeParam

# Reference resource
VPCId: !Ref MyVPC
InstanceId: !Ref MyInstance
```

**Intrinsic functions comparison:**
| Function | Returns |
|----------|--------|
| `Ref` | Parameter value or resource ID |
| `Fn::GetAtt` | Resource attributes |
| `Fn::Sub` | String substitution |
| `Fn::ImportValue` | Cross-stack value |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Fn::GetAtt | Returns attributes, không phải ID |
| C. Fn::Sub | String substitution |
| D. Fn::ImportValue | Import from other stacks |

---

### Question 8:
A CloudFormation stack update failed. What happens to the stack?

```
A. Stack is deleted
B. Stack automatically rolls back to previous working state
C. Stack remains in failed state
D. Stack continues with failed resources
```

### ✅ Answer: **B. Stack automatically rolls back to previous working state**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **CloudFormation Rollback behavior:**
  - **Stack Update fails**: Auto rolls back to previous working state
  - Can view logs for error messages
  - Resources deleted in reverse order

**Stack Creation vs Update:**
| Scenario | Behavior |
|----------|----------|
| Creation fails | Default: everything rolls back |
| **Update fails** | **Auto rolls back to previous state** |

**If rollback fails:**
- Fix resources manually
- Use `continue-update-rollback` API

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Deleted | Only if explicitly deleted |
| C. Remains failed | Can continue manually |
| D. Continues with failed | CloudFormation prevents this |

---

### Question 9:
What CloudFormation feature prevents resources from being **accidentally updated** during a stack update?

```
A. DeletionPolicy
B. Stack Policy
C. Termination Protection
D. Custom Resource
```

### ✅ Answer: **B. Stack Policy**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Stack Policy**:
  - JSON document defining update actions
  - **Protect resources** from unintentional updates
  - All resources **protected by default**
  - Specify explicit **ALLOW** for updatable resources

**Example:**
```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "Update:*",
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": "Update:*",
      "Resource": "AWS::RDS::DBInstance"
    }
  ]
}
```

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. DeletionPolicy | Controls what happens when resource is deleted |
| C. Termination Protection | Prevents stack deletion |
| D. Custom Resource | Custom provisioning logic |

---

### Question 10:
A CloudFormation template needs to reference a VPC ID exported by another stack. Which function should be used?

```
A. Fn::Ref
B. Fn::GetAtt
C. Fn::ImportValue
D. Fn::FindInMap
```

### ✅ Answer: **C. Fn::ImportValue**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **Fn::ImportValue**:
  - Import values **exported by other stacks**
  - Cross-stack references
  - Enables **collaboration** between teams

**Example:**
```yaml
# Stack 1 exports
Outputs:
  VPCId:
    Value: !Ref VPC
    Export:
      Name: my-vpc-id

# Stack 2 imports
Resources:
  Subnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !ImportValue my-vpc-id
```

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Fn::Ref | Reference within same stack |
| B. Fn::GetAtt | Get resource attributes |
| D. Fn::FindInMap | Get value from mapping |

---

### Question 11:
Which CloudFormation **DeletionPolicy** creates a snapshot before deleting a resource?

```
A. Delete
B. Retain
C. Snapshot
D. Backup
```

### ✅ Answer: **C. Snapshot**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **Snapshot policy**:
  - Creates **final snapshot** before deletion
  - For supported resources only

**Supported resources:**
| Resource | Snapshot Support |
|----------|-----------------|
| EBS Volume | ✅ |
| ElastiCache Cluster | ✅ |
| RDS DBInstance | ✅ |
| RDS DBCluster | ✅ |
| Redshift Cluster | ✅ |

**DeletionPolicy options:**
| Policy | Behavior |
|--------|----------|
| Delete (default) | Delete resource |
| Retain | Keep resource |
| **Snapshot** | **Create snapshot, then delete** |

---

## 4. AWS Messaging - SQS

### Question 12:
An application processes messages from an SQS queue. If the consumer fails to process a message, what happens to that message?

```
A. Message is deleted
B. Message reappears in the queue after visibility timeout
C. Message is sent to DLQ
D. Message is archived
```

### ✅ Answer: **B. Message reappears in the queue after visibility timeout**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Visibility Timeout**:
  - Message becomes invisible after being received
  - Default: **30 seconds**
  - If not deleted within timeout, message **reappears** in queue

**Flow:**
```
1. Message received (visible) ✓
2. Processing starts...
3. If timeout expires (no delete):
   → Message reappears in queue
   → Can be processed again
```

**Best practice:**
- Process message
- Delete message after successful processing
- Handle failures properly

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Deleted | Only if explicitly deleted |
| C. DLQ | Requires DLQ configuration |
| D. Archived | No automatic archiving |

---

### Question 13:
What is the **default message retention period** for Amazon SQS?

```
A. 1 day
B. 4 days
C. 7 days
D. 14 days
```

### ✅ Answer: **B. 4 days**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **SQS Message Retention:**
  - Default: **4 days**
  - Maximum: **14 days**
  - Minimum: 1 minute

**After retention expires:**
- Message automatically deleted
- Cannot be recovered

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. 1 day | Too short - default is 4 days |
| C. 7 days | Optional setting, not default |
| D. 14 days | Maximum, not default |

---

### Question 14:
Which SQS queue type guarantees **strict message ordering** and **exactly-once processing**?

```
A. Standard Queue
B. FIFO Queue
C. Priority Queue
D. Dead Letter Queue
```

### ✅ Answer: **B. FIFO Queue**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **FIFO Queue**:
  - **Strict ordering** (first-in-first-out)
  - **Exactly-once processing**
  - No duplicate messages
  - Throughput: 300 messages/second

**Standard vs FIFO:**
| Feature | Standard Queue | FIFO Queue |
|---------|---------------|-----------|
| Ordering | Best effort | **Strict** |
| Delivery | At least once | **Exactly once** |
| Duplicates | Can occur | **No duplicates** |
| Throughput | Unlimited | 300/s |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Standard Queue | At least once, best effort ordering |
| C. Priority Queue | Not an SQS type |
| D. Dead Letter Queue | For failed messages, not ordering |

---

## 5. AWS Messaging - SNS

### Question 15:
An application needs to **fan-out** messages to multiple downstream systems. Which AWS service combination is best?

```
A. SQS Queue
B. SNS Topic with multiple SQS subscriptions
C. Kinesis Data Streams
D. SNS Topic only
```

### ✅ Answer: **B. SNS Topic with multiple SQS subscriptions**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **SNS + SQS Fan-out Pattern**:
  - SNS Topic receives messages
  - Multiple SQS queues **subscribe** to topic
  - Each queue processes independently
  - **Decouples** producers from consumers

**Architecture:**
```
Publisher → SNS Topic
              ├── SQS Queue 1 → Consumer 1
              ├── SQS Queue 2 → Consumer 2
              └── SQS Queue 3 → Consumer 3
```

**Benefits:**
- Each consumer processes at own pace
- No coupling between consumers
- Failed consumers don't affect others

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. SQS only | Single consumer |
| C. Kinesis | For streaming, not fan-out |
| D. SNS only | No persistent storage |

---

## 6. Combined Scenarios

### Question 16:
A company wants to provision infrastructure using code that can be version-controlled. Which service should be used?

```
A. AWS CLI
B. AWS CloudFormation
C. AWS Elastic Beanstalk
D. AWS OpsWorks
```

### ✅ Answer: **B. AWS CloudFormation**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **CloudFormation**:
  - **Infrastructure as Code**
  - Version control (Git)
  - Declarative template
  - Provision AWS resources
  - Cost tracking

**Benefits:**
- No manual resource creation
- Changes reviewed through code
- Destroy and recreate infrastructure
- Cost estimation

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. AWS CLI | Interactive, not version-controlled |
| C. Elastic Beanstalk | Application deployment, not general infra |
| D. OpsWorks | Chef/Puppet-based, less common |

---

### Question 17:
An application needs to process messages asynchronously with **guaranteed ordering**. The message volume is low. Which combination is best?

```
A. SQS Standard Queue
B. SNS Topic
C. SQS FIFO Queue
D. Kinesis Data Streams
```

### ✅ Answer: **C. SQS FIFO Queue**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **SQS FIFO Queue**:
  - **Guaranteed strict ordering**
  - **Exactly-once processing**
  - Low volume is acceptable
  - Good for ordered processing

**When to use FIFO:**
- Message ordering critical
- No duplicates allowed
- Low to medium throughput

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Standard Queue | No ordering guarantee |
| B. SNS Topic | Push model, not queue |
| D. Kinesis | Overkill for low volume, designed for streaming |

---

### Question 18:
A Lambda function processes S3 events. The function sometimes fails to process. What should be implemented to handle failures?

```
A. Increase Lambda timeout
B. Configure SQS Dead Letter Queue
C. Increase Lambda memory
D. Disable SQS
```

### ✅ Answer: **B. Configure SQS Dead Letter Queue**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Dead Letter Queue (DLQ)**:
  - Failed messages sent to DLQ
  - For inspection and reprocessing
  - Prevents infinite retry loops

**Lambda with SQS DLQ:**
```yaml
# Configure on SQS queue
RedrivePolicy:
  deadLetterTargetArn: !GetAtt MyDLQ.Arn
  maxReceiveCount: 3
```

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Increase timeout | Addresses timeout, not general failures |
| C. Increase memory | Addresses memory issues, not failures |
| D. Disable SQS | Removes queue, doesn't handle failures |

---

### Question 19:
A company needs to deploy the same application to multiple AWS accounts. Which CloudFormation feature enables this?

```
A. Nested Stacks
B. StackSets
C. Cross-Stack References
D. Custom Resources
```

### ✅ Answer: **B. StackSets**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **StackSets**:
  - Create/update/delete stacks across **multiple accounts**
  - Across **multiple regions**
  - Single operation/template
  - Can apply to **AWS Organization**

**Use cases:**
- Compliance requirements across accounts
- Centralized infrastructure deployment
- Multi-account deployments

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Nested Stacks | Within same account |
| C. Cross-Stack | Between stacks in same account |
| D. Custom Resources | Custom provisioning logic |

---

### Question 20:
A web application needs to be deployed with zero downtime and the ability to quickly rollback. Which Elastic Beanstalk deployment is best?

```
A. All at once
B. Rolling
C. Immutable
D. Traffic Splitting
```

### ✅ Answer: **C. Immutable**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **Immutable deployment**:
  - **Zero downtime**
  - New instances in temporary ASG
  - Quickest rollback (terminate new ASG)
  - High cost but safest for production

**Deployment comparison:**
| Mode | Downtime | Rollback Speed | Cost |
|------|----------|---------------|------|
| All at once | Yes | Manual | None |
| Rolling | Partial | Manual | None |
| **Immutable** | **None** | **Instant** | High |
| Traffic Split | None | Auto | Medium |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. All at once | Has downtime |
| B. Rolling | Partial downtime, manual rollback |
| D. Traffic Splitting | Good but immutable fastest rollback |

---

## Summary Table

| Topic | Key Points |
|-------|------------|
| **CloudFront Signed URLs** | Premium content, time-limited access |
| **CloudFront Invalidation** | Force cache refresh |
| **EB Deployments** | Immutable = fastest rollback |
| **EB .ebextensions** | YAML/JSON, option_settings |
| **CloudFormation Ref** | Returns parameter value or resource ID |
| **CloudFormation Rollback** | Auto on update failure |
| **Stack Policy** | Protect resources from updates |
| **Fn::ImportValue** | Cross-stack references |
| **DeletionPolicy Snapshot** | Creates snapshot before delete |
| **SQS Visibility Timeout** | Message reappears if not deleted |
| **SQS Retention** | Default 4 days, max 14 days |
| **SQS FIFO** | Strict ordering, exactly-once |
| **SNS + SQS Fan-out** | Decouple multiple consumers |
| **StackSets** | Multi-account, multi-region |
