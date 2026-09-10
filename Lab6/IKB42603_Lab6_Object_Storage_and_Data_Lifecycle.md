## Course Information
---
*Course:* IKB42603 Cloud Computing Security Essentials
*Lab:* Lab 0 - Environment Setup
*Name:* MUHAMMAD AMEER BIN IDRIS
*Date:* 30 July 2026

# IKB42603 Lab 6: Object Storage & Data Lifecycle

## Session A (Week 11) — Object Storage & the Exposure Problem

### One-Time Environment Setup
Start a clean, activated LocalStack instance and point the CLI at it.
![start clean](1.%20start%20clean.png)
![Point the CLI at LocalStack](2.%20Point%20the%20CLI%20at%20LocalStack.png)

### Task 1 — Classify the Data Before You Store It
Create a bucket for a hospital records system and store three objects of different sensitivity, tagging each with its classification.
![Classify the Data Before You Store It](3.%20Classify%20the%20Data%20Before%20You%20Store%20It.png)
![Classify the Data Before You Store It](4.%20Classify%20the%20Data%20Before%20You%20Store%20It.png)
![Classify the Data Before You Store It](5.%20Classify%20the%20Data%20Before%20You%20Store%20It.png)
![Classify the Data Before You Store It](6.%20Classify%20the%20Data%20Before%20You%20Store%20It.png)

### Task 2 — Reproduce the Archetypal Breach
Build a policy deliberately naming "Principal": "*", then read your own confidential record with no credentials at all.
![Reproduce the Archetypal Breach](7.%20Reproduce%20the%20Archetypal%20Breach.png)
![no AWS credentials-no CLI,-just a URL](8.%20no%20AWS%20credentials-no%20CLI,-just%20a%20URL.png)

### Task 3 — Remediate with Block Public Access
Apply the account-level guardrail to the bucket to prevent the bucket from becoming public.
![Remove the offending policy](9.%20Remove%20the%20offending%20policy.png)
![Apply the account-level guardrail to the bucket](10.%20Apply%20the%20account-level%20guardrail%20to%20the%20bucket.png)
![Try to re-introduce the public policy](11.%20Try%20to%20re-introduce%20the%20public%20policy%20-%20the%20guardrail%20should%20refuse%20it.png)
![Re-test the anonymous read](12.%20Re-test%20the%20anonymous%20read.png)
![read access for your own account only](13.%20read%20access%20for%20your%20own%20account%20only.png)

### Task 4 — Identity Policy vs Resource Policy
When Identity and Resource policies disagree, an explicit Deny always wins.
![An analyst whose IAM policy allows reading everything](14.%20An%20analyst%20whose%20IAM%20policy%20allows%20reading%20everything.png)
![Note both values](15.%20Note%20both%20values%20-%20you%20will%20paste%20them%20into%20a%20named%20profile.png)
![Copy the two values into these variables](16.%20Copy%20the%20two%20values%20into%20these%20variables%20-%20keep%20the%20quotes.png)
![Now the bucket owner disagrees about one prefix](17.Now%20the%20bucket%20owner%20disagrees%20about%20one%20prefix.png)
![Should SUCCEED](18.%20Should%20SUCCEED%20-%20allowed%20by%20both%20policies.png)
![Should FAIL](19.%20Should%20FAIL%20-%20IAM%20allows,%20but%20the%20bucket%20policy%20explicitly%20denies.png)

## Session B (Week 12) — Protecting, Retaining and Retiring Data

### Task 5 — Default Encryption at Rest (SSE-KMS)
Make encryption a property of the bucket, so that every object is encrypted whether or not the developer remembers to ask.
![A dedicated key for this bucket](20.%20A%20dedicated%20key%20for%20this%20bucket.png)
![Upload with NO encryption flags at all](21.%20Upload%20with%20NO%20encryption%20flags%20at%20all.png)

### Task 6 — Delegated Access and the Condition-Key Trap
Use a presigned URL to grant a specific action on a specific object for a limited time.
![Time-bounded-signed- single-object access](22.%20Time-bounded-signed-%20single-object%20access.png)
![Paste the URL into the variable](23.Paste%20the%20URL%20into%20the%20variable%20-%20keep%20the%20quotes.png)
![Wait for it to lapse- then try the same url again](24.%20Wait%20for%20it%20to%20lapse-%20then%20try%20the%20same%20url%20again.png)
![Any ordinary call - expect it to be refused](25.%20Any%20ordinary%20call%20-%20expect%20it%20to%20be%20refused.png)
![Recover before continuing](26.%20Recover%20before%20continuing.png)

### Task 7 — Versioning, Delete Markers & Data Remanence
With versioning enabled, delete does not delete. It writes a delete marker over the top and every prior version survives underneath.
![showed remanence inside a container volume](27.%20showed%20remanence%20inside%20a%20container%20volume.png)
![Two more revisions of the same record](28.%20Two%20more%20revisions%20of%20the%20same%20record.png)
!['delete' the record](29.%20'delete'%20the%20record%20.png)
![A delete marker is now the current version](30.%20A%20delete%20marker%20is%20now%20current%20version.png)
![To an ordinary reader the object is gone](31.%20To%20an%20ordinary%20reader%20the%20object%20is%20gone.png)
![unredacted record is still there](32.%20unredacted%20record%20is%20still%20there.png)
![Permanent-per-version deletion](33.%20Permanent-per-version%20deletion.png)

### Task 8 — Lifecycle, Retention & Cryptographic Erasure
A lifecycle configuration is the automated, auditable expression of your retention policy.
![A lifecycle configuration is the automated_auditable expression of your retention policy](34.%20A%20lifecycle%20configuration%20is%20the%20automated_auditable%20expression%20of%20your%20retention%20policy.png)
![Destroy the key and the ciphertext becomes unrecoverable noise](35.%20Destroy%20the%20key%20and%20the%20ciphertext%20becomes%20unrecoverable%20noise.png)
![Attempt to read an object encrypted under the disabled key](36.%20Attempt%20to%20read%20an%20object%20encrypted%20under%20the%20disabled%20key.png)

### Cleanup & Teardown
![Cleanup & Teardown](38.%20Cleanup%20&%20Teardown.png)
![remove every version explicitly](39.%20remove%20every%20version%20explicitly.png)

---

## Deliverables & Assessment

### 2. Data Classification Table
| Classification | Who may read it | Impact if leaked | Control you will apply |
|---|---|---|---|
| public | Anyone / Public | Low / None | Block Public Access (Guardrail) |
| internal | Internal Staff | Medium | Resource-based Policy (Least Privilege) |
| confidential | Specific Authorized Personnel | High / Critical | Default Encryption (SSE-KMS), Versioning & Lifecycle Management |

### 3. Short-Answer Questions

**Q: Which single element of the Task 2 policy caused the exposure, and why is Principal: "*" more dangerous on a bucket policy than an over-broad IAM policy attached to one user?**
**A:** The single element that caused the exposure was `"Principal": "*"`. It is highly dangerous on a bucket policy because it grants access to anyone on the internet, including completely anonymous users without AWS credentials. An over-broad IAM policy only grants access to the specific authenticated user it is attached to.

**Q: Explain the difference between an identity-based policy and a resource-based policy. In Task 4, which one decided each of the analyst's two requests?**
**A:** An identity-based policy is attached to an IAM identity (user/role) and defines what they can do. A resource-based policy is attached to a resource (like an S3 bucket) and defines who can access it. In Task 4, the read request for `internal` was evaluated and allowed primarily by the identity-based policy. The read request for `confidential` was explicitly denied by the resource-based policy, overriding the identity-based allow.

**Q: Block Public Access is described as a guardrail rather than a control. What is the difference, and why does the distinction matter for an organisation with many engineers?**
**A:** A control (like an IAM policy) explicitly grants or denies access. A guardrail operates globally at a higher level, enforcing strict boundaries that override unsafe configurations. For an organization with many engineers, a guardrail ensures that even if a developer makes a mistake and tries to apply a public bucket policy, it will be automatically blocked, acting as a fail-safe mechanism against misconfigurations.

**Q: Your bucket has default SSE-KMS encryption. Does that protect the confidential record from the analyst in Task 4? Explain precisely what server-side encryption does and does not defend against.**
**A:** No, SSE-KMS does not protect the record if the analyst has IAM permissions to read the S3 object and use the KMS key. Server-side encryption defends against unauthorized access to the underlying physical storage media (e.g., if hard drives are compromised) and allows for cryptographic erasure. It does not defend against unauthorized access at the application/API layer if a user is granted the required logical permissions.

**Q: A patient invokes their right to erasure. Using your Task 7 evidence, explain why delete-object alone is not compliant, and describe two mechanisms that would make the deletion provable.**
**A:** With versioning enabled, `delete-object` alone merely creates a delete marker while retaining the original data underneath (data remanence), which means the data is not actually erased. Two mechanisms for provable deletion are:
1. **Permanent per-version deletion**: Explicitly specifying the `version-id` in the delete request.
2. **Cryptographic erasure**: Deleting the KMS encryption key that protects the data, rendering all versions unrecoverable.

**Q: You are the auditor in Week 11. Name three commands from this lab whose output you would collect as compliance evidence, and state which control each one evidences.**
**A:**
1. `aws s3api get-public-access-block` - Evidences the guardrail control preventing public exposure.
2. `aws s3api get-bucket-encryption` - Evidences the control for default encryption at rest.
3. `aws s3api get-bucket-lifecycle-configuration` - Evidences the control for automated data retention/retirement policies.

### 4. Verification Command
![prove the bucket's final security posture](37.%20prove%20the%20bucket's%20final%20security%20posture.png)
