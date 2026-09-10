## Course Information
---
*Course:* IKB42603 Cloud Computing Security Essentials
*Lab:* Lab 0 - Environment Setup
*Name:* MUHAMMAD AMEER BIN IDRIS
*Date:* 30 July 2026

# IKB42603 Lab 6: Object Storage & Data Lifecycle

## Session A (Week 11) — Object Storage & the Exposure Problem

### One-Time Environment Setup
To begin the lab, we start a clean, activated LocalStack instance and point our AWS CLI towards it. We also enforce IAM evaluation (`ENFORCE_IAM=1`) so that the LocalStack environment respects Identity and Access Management policies appropriately.
<img src="1. start clean.png" alt="start clean" />
<img src="2. Point the CLI at LocalStack.png" alt="Point the CLI at LocalStack" />

### Task 1 — Classify the Data Before You Store It
Security decisions should always follow data classification. In this task, we create an S3 bucket for a hospital records system. We then store three text objects of varying sensitivity and tag each with its proper classification: `public`, `internal`, and `confidential`. These tags provide the foundation for applying granular security controls later.
<img src="3. Classify the Data Before You Store It.png" alt="Classify the Data Before You Store It" />
<img src="4. Classify the Data Before You Store It.png" alt="Classify the Data Before You Store It" />
<img src="5. Classify the Data Before You Store It.png" alt="Classify the Data Before You Store It" />
<img src="6. Classify the Data Before You Store It.png" alt="Classify the Data Before You Store It" />

### Task 2 — Reproduce the Archetypal Breach
The most common cause of cloud data leaks is an overly permissive bucket policy. We deliberately reproduce this vulnerability by applying a bucket policy with `"Principal": "*"`. This allows anyone on the internet to read the contents. We confirm the breach by anonymously fetching the `confidential` record via a simple `curl` request, proving the data is exposed without any AWS credentials.
<img src="7. Reproduce the Archetypal Breach.png" alt="Reproduce the Archetypal Breach" />
<img src="8. no AWS credentials-no CLI,-just a URL.png" alt="no AWS credentials-no CLI,-just a URL" />

### Task 3 — Remediate with Block Public Access
To fix the exposure, we first delete the offending public policy. However, to prevent this from happening again in the future, we apply an account-level guardrail called **Block Public Access**. This feature blocks any future attempts to apply public policies or ACLs. Finally, we replace the policy with a least-privilege approach, restricting read access strictly to our own account and scoped only to the `internal/` prefix.
<img src="9. Remove the offending policy.png" alt="Remove the offending policy" />
<img src="10. Apply the account-level guardrail to the bucket.png" alt="Apply the account-level guardrail to the bucket" />
<img src="11. Try to re-introduce the public policy - the guardrail should refuse it.png" alt="Try to re-introduce the public policy" />
<img src="12. Re-test the anonymous read.png" alt="Re-test the anonymous read" />
<img src="13. read access for your own account only.png" alt="read access for your own account only" />

### Task 4 — Identity Policy vs Resource Policy
Cloud storage access is evaluated by looking at both the **Identity Policy** (IAM) attached to the user and the **Resource Policy** (Bucket Policy) attached to the storage. When they conflict, an explicit `Deny` always wins. We create an IAM user ("DataAnalyst") whose policy allows reading everything. However, we add a bucket policy that explicitly denies them access to the `confidential/` prefix. As a result, the analyst successfully reads the `internal/` record but is explicitly blocked from the `confidential/` record.
<img src="14. An analyst whose IAM policy allows reading everything.png" alt="An analyst whose IAM policy allows reading everything" />
<img src="15. Note both values - you will paste them into a named profile.png" alt="Note both values" />
<img src="16. Copy the two values into these variables - keep the quotes.png" alt="Copy the two values into these variables" />
<img src="17.Now the bucket owner disagrees about one prefix.png" alt="Now the bucket owner disagrees about one prefix" />
<img src="18. Should SUCCEED - allowed by both policies.png" alt="Should SUCCEED" />
<img src="19. Should FAIL - IAM allows, but the bucket policy explicitly denies.png" alt="Should FAIL" />

## Session B (Week 12) — Protecting, Retaining and Retiring Data

### Task 5 — Default Encryption at Rest (SSE-KMS)
To ensure that all data is encrypted automatically, we configure default Server-Side Encryption (SSE-KMS) for the bucket using a customer-managed KMS key. When a new file is uploaded without explicitly passing any encryption flags, the bucket automatically encrypts it. This guarantees data protection at rest, even if the uploader forgets to request it.
<img src="20. A dedicated key for this bucket.png" alt="A dedicated key for this bucket" />
<img src="21. Upload with NO encryption flags at all.png" alt="Upload with NO encryption flags at all" />

### Task 6 — Delegated Access and the Condition-Key Trap
When we need to securely share a specific object with someone who doesn't have AWS credentials, we generate a **Presigned URL**. This grants time-limited access to the object. After the expiration time passes, the URL becomes invalid. We then test a bucket policy condition (`aws:SecureTransport`) that enforces TLS (HTTPS). Since LocalStack handles HTTP locally, this condition traps our requests and denies them, showing how environmental condition keys work.
<img src="22. Time-bounded-signed- single-object access.png" alt="Time-bounded-signed- single-object access" />
<img src="23.Paste the URL into the variable - keep the quotes.png" alt="Paste the URL into the variable" />
<img src="24. Wait for it to lapse- then try the same url again.png" alt="Wait for it to lapse" />
<img src="25. Any ordinary call - expect it to be refused.png" alt="Any ordinary call - expect it to be refused" />
<img src="26. Recover before continuing.png" alt="Recover before continuing" />

### Task 7 — Versioning, Delete Markers & Data Remanence
Object storage systems handle deletion differently when versioning is enabled. Instead of destroying the data, deleting an object places a "delete marker" on top, hiding it from standard read requests. However, the original unredacted versions still exist underneath (data remanence). To comply with privacy laws (e.g., right to erasure), we must perform permanent, per-version deletion using the specific version ID to actually destroy the data.
<img src="27. showed remanence inside a container volume.png" alt="showed remanence inside a container volume" />
<img src="28. Two more revisions of the same record.png" alt="Two more revisions of the same record" />
<img src="29. 'delete' the record .png" alt="'delete' the record" />
<img src="30. A delete marker is now the current version.png" alt="A delete marker is now the current version" />
<img src="31. To an ordinary reader the object is gone.png" alt="To an ordinary reader the object is gone" />
<img src="32. unredacted record is still there.png" alt="unredacted record is still there" />
<img src="33. Permanent-per-version deletion.png" alt="Permanent-per-version deletion" />

### Task 8 — Lifecycle, Retention & Cryptographic Erasure
Deleting versions manually does not scale well. Instead, we implement an automated **Lifecycle Configuration** to define our retention policy (e.g., expiring old versions automatically). Finally, we demonstrate the fastest deletion method available in the cloud: **Cryptographic Erasure**. By scheduling the KMS key used for encryption for deletion, all objects encrypted under that key are instantly rendered as unrecoverable noise, ensuring provable destruction.
<img src="34. A lifecycle configuration is the automated_auditable expression of your retention policy.png" alt="A lifecycle configuration" />
<img src="35. Destroy the key and the ciphertext becomes unrecoverable noise.png" alt="Destroy the key" />
<img src="36. Attempt to read an object encrypted under the disabled key.png" alt="Attempt to read an object encrypted under the disabled key" />

### Cleanup & Teardown
To fully delete a versioned bucket, we cannot use a simple force delete command. We must explicitly list and remove every object version and delete marker first before the bucket becomes genuinely empty and can be destroyed.
<img src="38. Cleanup & Teardown.png" alt="Cleanup & Teardown" />
<img src="39. remove every version explicitly.png" alt="remove every version explicitly" />

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
<img src="37. prove the bucket's final security posture.png" alt="prove the bucket's final security posture" />
