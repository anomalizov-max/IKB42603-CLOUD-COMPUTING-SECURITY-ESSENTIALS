## Course Information
---
**Course:** IKB42603 Cloud Computing Security Essentials

**Lab:** Lab 5: Monitoring, Logging & Incident Detection

**Name:** MUHAMMAD AMEER BIN IDRIS

**Student ID:** 52215124748

**Date:** 04 September 2026

## Lab Learning Outcomes
At the end of this lab, the objectives were to:
1. Collect and centralise logs from multiple services (cloud telemetry).
2. Distinguish logs from events and query logs for security-relevant activity.
3. Build a tamper-evident (hash-chained) log and detect alteration.
4. Detect an incident by correlating events (e.g. brute-force followed by a suspicious action).
5. Execute the incident-response steps: detect, contain, collect evidence, and document a timeline.

---

## Session A (Week 9) — Logging & Centralisation

### Setup — Start LocalStack
To begin the lab, we instantiated a local mock AWS cloud environment using LocalStack within a Docker container. This establishes an endpoint (port 4566) mimicking actual AWS cloud capabilities. Then, utilizing the AWS CLI, we successfully initialized a new CloudWatch log group (`/ccse/app`) and a designated `auth` log stream, providing us with the foundation needed for cloud-based telemetry storage.

![1. Setup - Start LocalStack](Evidence/1.%20Setup%20-%20Start%20LocalStack.png)
![2. Start LocalStack](Evidence/2.%20Start%20LocalStack.png)

### Task 1 — Generate Application Logs
In this task, we simulated a realistic authentication sequence by writing a custom structured text log into an `auth.log` file. This log includes a normal login, but more importantly, it tracks a suspected threat actor systematically probing the application from a remote IP (`203.0.113.9`). By manually assembling events across a targeted timeline, we emulated how typical applications record both failed and successful user interactions along with highly sensitive actions like massive data exports (e.g., `500MB`). This laid the groundwork for event correlation in later tasks.

![3. Generate Application Logs](Evidence/3.%20Generate%20Application%20Logs.png)

### Task 2 — Centralise Logs (Ship to CloudWatch)
Keeping security events securely backed up is a critical practice to avoid manipulation by an attacker. Rather than retaining the raw `auth.log` unprotected precisely where an application executes, we iteratively pushed each authentication record directly into our previously established LocalStack CloudWatch Logs stream. Applying the CLI's `put-log-events` command simulated translating unmanaged, localized plain-text metrics into durable cloud-based telemetry that can be safely preserved and heavily queried on demand. We further verified this by retrieving the deployed logs back using the `get-log-events` directive.

![4. Centralise Logs](Evidence/4.%20Centralise%20Logs.png)
![5. Read them back from the central store](Evidence/5.%20Read%20them%20back%20from%20the%20central%20store.png)

### Task 3 — Query for Security-Relevant Activity
With our centralized architecture populated, we employed standard Linux shell querying mechanisms (`grep`, `awk`, and `uniq`) to isolate anomalies hidden deeply inside the dataset. By explicitly parsing strings looking only for authentication rejections (`LOGIN_FAIL`) and aggregating these hits by source IP address, we distilled raw records into actionable situational awareness. We were able to separate the "signal from the noise," confirming that the external user probing the administrative portal wasn't just experiencing typing errors, but was actively executing a sustained brute-force attempt.

---

## Session B (Week 10) — Tamper-Proofing, Detection & Response

### Task 4 — Tamper-Proof (Hash-Chained) Logs
An accomplished adversary's first post-exploitation objective is often modifying audit trails (e.g., downgrading an export record from 500MB to merely 5MB to evade data loss alarms). To combat this, we enforced security auditing integrity by structuring the historical log entries into a cryptographic hash chain. Utilizing SHA256 algorithms, each new event mathematically intertwines with the checksum of the preceding entry. Modifying a completed entry logically fractures the sequence at the point of alteration, providing concrete forensic proof that the environment was compromised and generating a mismatched final output hash.

![6. Tamper-Proof (Hash-Chained) Logs](Evidence/6.%20Tamper-Proof%20(Hash-Chained)%20Logs.png)
![7. change the EXPORT size, re-verify, and watch the chain break](Evidence/7.%20change%20the%20EXPORT%20size,%20re-verify,%20and%20watch%20the%20chain%20break.png)

### Task 5 — Detect the Incident (Correlation)
In isolation, no single activity independently screams absolute danger; failing passwords happen to normal individuals frequently, and successful administrative exports are similarly expected actions. However, we simulated how modern SIEM (Security Information and Event Management) tools function by aggressively correlating chronologies. We mapped a tight timeline tracking repeated `LOGIN_FAIL` variables mapped strictly against the threat actor's IP, seamlessly linking them sequentially to an authorized `LOGIN_OK` and an unprecedented `EXPORT_DATA` command. Measuring these concurrent dependencies generated a high-priority, automated incident response alert for "probable brute-force -> compromise -> data exfiltration."

![8. Detect the Incident](Evidence/8.%20Detect%20the%20Incident.png)
![9. Detect the Incident](Evidence/9.%20Detect%20the%20Incident.png)

### Task 6 — Incident Response
Once the automated correlation systems firmly validated the malicious intrusion, we simulated emergency incident response handling.
**Containment (IP Block via iptables):**
We aggressively severed ongoing external connectivity and contained the exfiltration leak by executing an active `iptables` drop rule within a Docker environment, completely blackholing additional ingress traffic matching the offending IP (`203.0.113.9`).

![10. block the attacker IP (model with an iptables rule)](Evidence/10.%20block%20the%20attacker%20IP%20(model%20with%20an%20iptables%20rule).png)

**Evidence Collection:**
Simultaneously, we initiated forensic preservation operations. We firmly cloned the remaining uncompromised original authentication logs directly into an immutable, timestamped file structure while concurrently storing a verified one-way SHA256 encryption checksum to provide unquestioned proof regarding integrity before continuing further remediation processes.

![11. make an immutable, timestamped evidence copy with its hash](Evidence/11.%20make%20an%20immutable,%20timestamped%20evidence%20copy%20with%20its%20hash.png)

---

## Deliverables & Assessment

### 2. Incident Report
* **Detection:** Detected based on the correlation of successive `LOGIN_FAIL` logs across a single IP address followed immediately by an authenticated `LOGIN_OK` and a significant `EXPORT_DATA` operation.
* **Analysis:** The attacker (IP `203.0.113.9`) performed a dictionary or brute-force attack to capture the `admin` credential. Once in, 500MB of sensitive data was maliciously exported. Tamper detection correctly flagged simulated attempts to alter this evidence trail.
* **Containment:** Deployed an `iptables` drop rule explicitly terminating all incoming traffic (`INPUT` chain) from the threat actor's IP (`203.0.113.9`), sealing the active data leak.
* **Evidence & integrity:** Captured timestamp-locked copies of the `auth.log` data to a new audit file alongside calculating its final unalterable SHA256 checksum so that forensic and legal validations remain intact.
* **Lesson learned:** Centralised logging and hash chaining are critical because prevention models eventually fail. Correlating data points provides actionable warnings that localized analysis ordinarily misses.

### 3. Short-Answer Questions
**Q1. What is the difference between a log and an event? Give an example of each from this lab.**
A **log** is a durable, permanent historical recording of an application's activity sent to a file or stream. Example: `2025-03-01T09:01:10 LOGIN_FAIL user=admin ip=203.0.113.9`.
An **event** is an actionable trigger dynamically generated in near real-time, typically based on a pattern found within logs. Example: The output alert string: `ALERT: probable brute-force -> compromise -> data exfiltration`.

**Q2. Why must audit logs be tamper-proof, and how does a hash chain achieve this?**
Audit records must be tamper-proof because modern attackers prioritize editing logs directly to obscure their activities effectively erasing indicators of compromise. A hash chain establishes tamper-proofing by piping each string (combined tightly with the prior line's hash) into a one-way hashing function (like SHA256). Attempting to edit a single past character definitively breaks the sequence and generates an inaccurate final checksum.

**Q3. How did correlation detect an incident that no single log line revealed?**
An individual log line noting an authentication failure is typically benign (e.g., poor typing), while a successful export represents normal administrative behavior. However, fusing these disparate records together across a specific IP mapping over an abbreviated timeframe painted a wider narrative exposing a successful brute-force infiltration ending directly in unapproved data exfiltration.

**Q4. List the incident-response steps you performed and the goal of each.**
1. **Contain**: Using `iptables` to drop the inbound packets originating from the hostile IP, neutralizing the network threat and preventing further extraction of data. 
2. **Collect Evidence**: Archiving the compromised server's auth logs utilizing immutable timestamped copies and maintaining forensic reliability by computing its SHA256 cryptographic hash.
3. **Document**: Fostered accountability and structured context by synthesizing the attack timeline directly into an Incident Report covering the containment methodology and discovered anomalies.

**Q5. How do the same logs serve both security monitoring and compliance evidence (Weeks 6, 11)?**
Logs centrally shipped (e.g. CloudWatch) allow real-time SIEM systems to scrutinize incoming telemetry data dynamically, identifying live threats immediately to deploy automated defensive measures. Concurrently, leveraging hash chains offers the durability ensuring that the identical historical datasets function as undeniable legal proof aligning tightly with organizational and governmental compliance specifications.

---

## Verification Command
Executed the final verification commands successfully to fetch CloudWatch log statuses and compare final SHA256 signature chains.

![12. Verification Command](Evidence/12.%20Verification%20Command.png)

## Cleanup & Teardown

![13. Cleanup & Teardown](Evidence/13.%20Cleanup%20&%20Teardown.png)
