# 4.0 Data and Database Security — 23%

Second-highest weight. This domain covers encryption, governance, access control, physical/logical security, and attack types. Expect scenario questions on authentication policies and conceptual questions on everything else.

---

## 4.1 Explain Data Security Concepts

### Encryption

Encryption converts readable data (plaintext) into unreadable ciphertext using a cryptographic algorithm and key. Know the two states:

#### Data in Transit

Data moving across a network between client and server or between servers.

| Method | Where Encryption Happens | Example |
|--------|--------------------------|---------|
| **Client-side encryption** | Data is encrypted by the application before it leaves the client | App encrypts a credit card number before sending it to the API |
| **In-transit encryption** | The transport channel itself is encrypted | TLS/SSL between application and database server |
| **Server-side encryption** | The server encrypts data upon receipt before storing it | Cloud provider encrypts data after it arrives at the storage layer |

**Exam tip:** A question that says "data must be protected while traveling between the web server and the database" is asking about **in-transit encryption** (TLS). If it says "the application must encrypt data before sending," that's **client-side encryption**.

#### Data at Rest

Data stored on disk — database files, backups, logs.

- **Transparent Data Encryption (TDE)** — encrypts database files at the storage level; queries see plaintext because decryption happens in memory.
- **Column-level encryption** — encrypts specific sensitive columns; more granular but higher application complexity.
- **Full-disk encryption** — OS-level encryption of the entire volume (BitLocker, LUKS).

### Data Masking

- Replaces sensitive data with realistic but fictitious values (e.g., SSN `123-45-6789` → `XXX-XX-6789`).
- **Static masking** — permanently alters data in non-production copies.
- **Dynamic masking** — masks data at query time based on user role; production data remains unchanged.

#### Data Discovery

- The process of scanning databases to identify where sensitive data resides (PII, PHI, cardholder data).
- Must happen before masking or classification — you can't protect what you don't know about.

### Data Destruction Techniques

When data must be permanently removed (end-of-life, compliance requirement):

- **Logical deletion** — mark records as deleted (soft delete); data is recoverable.
- **Physical deletion** — `DELETE` or `TRUNCATE`; data may still be recoverable from backups or disk forensics.
- **Cryptographic erasure** — destroy the encryption key, rendering encrypted data permanently unreadable.
- **Media sanitization** — degaussing, overwriting, or physical destruction of storage media.

### Data Security Audit

| Audit Focus | What to Check |
|-------------|---------------|
| **Expired accounts** | Disable or remove accounts that are no longer in use; attackers target dormant accounts |
| **Connection requests** | Review logs for unauthorized or anomalous connection attempts |

### Code Auditing

| Area | What to Look For |
|------|-----------------|
| **SQL code** | SQL injection vulnerabilities, dynamic SQL without parameterization, excessive permissions |
| **Credential storage checks** | Passwords stored in plaintext, hardcoded connection strings, secrets in source control |

**What they might ask:** "During a security audit, the DBA finds a service account with a password that hasn't been changed in two years — what should they do?" → Rotate the password immediately and enforce a password rotation policy.

---

## 4.2 Explain the Purpose of Governance and Regulatory Compliance

### Data Loss Prevention (DLP)

- Policies and tools that detect and prevent unauthorized transmission of sensitive data.
- Examples: blocking email attachments containing SSNs, preventing database exports to unauthorized locations, monitoring clipboard and USB activity.

### Data Retention Policies

- Define how long each data category must be kept and when it must be deleted.
- Driven by business needs and legal/regulatory requirements.
- Over-retention increases risk and storage costs; under-retention causes compliance violations.

### Data Classification

Classify data by sensitivity to apply appropriate controls:

| Classification | Examples | Typical Controls |
|----------------|----------|-----------------|
| **PII** (Personally Identifiable Information) | Name, SSN, email, address, date of birth | Encryption, masking, access restrictions, breach notification |
| **PHI** (Personal Health Information) | Medical records, diagnoses, insurance IDs | HIPAA-mandated safeguards, audit trails, minimum necessary access |
| **PCI DSS** (Payment Card Industry Data Security Standard) | Cardholder name, card number, CVV, expiration | Encryption of cardholder data, network segmentation, quarterly scans, access logging |

### Global and Regional Regulations

| Regulation | Scope | Key Requirements |
|------------|-------|-----------------|
| **GDPR** (General Data Protection Regulation) | EU/EEA residents' data, regardless of where the organization is based | Right to erasure, data portability, breach notification within 72 hours, lawful basis for processing, Data Protection Officer (DPO) |
| **Regional regulations** | State, provincial, or sector-specific laws (e.g., CCPA, PIPEDA, HIPAA) | Vary by jurisdiction; the exam uses "regional regulations" as a catch-all — know that compliance requirements differ by geography |

**Exam tip:** If a question mentions a user requesting deletion of their personal data, think **GDPR — right to erasure**. If it mentions cardholder data, think **PCI DSS**.

---

## 4.3 Given a Scenario, Implement Authentication and Authorization Policies

This is scenario-based — expect "Given this situation, what should the DBA configure?"

### Access Controls

| Concept | Definition | Implementation |
|---------|------------|----------------|
| **Rights and privileges** | Permissions granted to users or roles (`SELECT`, `INSERT`, `EXECUTE`, `ALTER`, etc.) | Grant at the most granular level practical; use roles to simplify management |
| **Least privilege** | Every user/account gets only the permissions required to perform their job — nothing more | Default deny; explicitly grant needed permissions; review periodically |

**Decision steps for scenario questions:**

1. **What does the user need to do?** Read data → `SELECT` only. Modify data → `INSERT`/`UPDATE`/`DELETE`. Manage schema → DDL permissions.
2. **Apply least privilege.** Never grant `db_owner` or `sysadmin` for routine tasks.
3. **Use roles.** Group users by function (e.g., `read_only_analysts`, `app_service_account`) rather than granting permissions individually.

### Password Policies

| Policy Element | Recommendation |
|----------------|---------------|
| **Complexity** | Minimum length, mix of character types |
| **Expiration** | Regular rotation (e.g., every 90 days) for privileged accounts |
| **History** | Prevent reuse of recent passwords |
| **Lockout** | Lock after N failed attempts to thwart brute-force attacks |

### Service Accounts

- Dedicated accounts used by applications or automated processes to connect to the database.
- Should have minimal privileges scoped to the specific application's needs.
- **Do not** use interactive-login credentials as service accounts.
- Rotate service-account passwords on a schedule; store them in a secrets manager, never in source code.

### Identity and Access Management (IAM)

- Centralized framework for managing digital identities and their access rights.
- Integration with directory services (Active Directory, LDAP) or cloud IAM (AWS IAM, Azure AD).
- Enables single sign-on (SSO), multi-factor authentication (MFA), and centralized audit trails.

**What they might ask:** "A new analyst needs to run reports but should not be able to modify data — what should the DBA do?" → Create or assign a read-only role with `SELECT` permissions on the relevant schemas; apply least privilege.

---

## 4.4 Explain the Purpose of Database Infrastructure Security

### Physical Security

Physical access to database servers must be controlled. If an attacker has physical access, all logical controls can be bypassed.

| Control | Purpose |
|---------|---------|
| **Access control** | Locked server rooms, badge readers, mantraps |
| **Biometrics** | Fingerprint, retinal scan — high-assurance identity verification for data-center entry |
| **Surveillance** | CCTV cameras monitoring server rooms and entry points; recorded for forensic review |
| **Fire suppression** | Clean-agent suppression (FM-200, Novec) that won't damage electronics; smoke detection and alarms |
| **Cooling system** | HVAC and precision cooling to maintain safe operating temperatures; overheating causes hardware failure and outages |

### Logical Security

| Control | Purpose |
|---------|---------|
| **Firewall** | Filters network traffic by source/destination IP, port, and protocol; only allow connections from authorized application servers |
| **Perimeter network (DMZ)** | Network segment between the external network and the internal network; web servers sit here, databases do **not** |
| **Port security** | Restrict which ports are open on the DB server; change default ports where practical; disable unused services and ports |

**Exam tip:** The database should **never** reside in the DMZ. It belongs on an internal network segment, accessible only from the application tier through the firewall.

---

## 4.5 Describe Types of Attacks and Their Effects on Data Systems

This is a "Describe" objective — know the attack type, how it works, and its effect. Expect "which attack does this describe?" questions.

| Attack | How It Works | Effect on Data Systems | Mitigation |
|--------|-------------|----------------------|------------|
| **SQL injection** | Attacker inserts malicious SQL into input fields (e.g., login forms) that get executed by the database | Unauthorized data access, data modification or deletion, privilege escalation, full database compromise | Parameterized queries / prepared statements; input validation; least-privilege DB accounts; WAF |
| **Denial of Service (DoS)** | Overwhelms the server with traffic or resource-intensive requests so legitimate users can't connect | Database unavailability, dropped connections, service outage | Rate limiting, connection throttling, firewall rules, load balancing, auto-scaling |
| **On-path attacks** (man-in-the-middle) | Attacker intercepts communication between client and database server | Credential theft, data interception, session hijacking | TLS/SSL encryption for all DB connections; certificate pinning; mutual authentication |
| **Brute-force attacks** | Automated repeated login attempts with different passwords until one succeeds | Unauthorized access if successful; account lockouts if policies are in place | Account lockout policies, strong password requirements, MFA, monitoring failed logins |
| **Phishing** | Social engineering via fake emails/websites to trick users into revealing credentials | Compromised DBA or user accounts → unauthorized database access | Security awareness training, MFA, email filtering, verifying sender identity |
| **Malware — Ransomware** | Encrypts database files and demands payment for the decryption key | Complete data loss or extended downtime if backups are unavailable or also encrypted | Offline/immutable backups, endpoint protection, network segmentation, principle of least privilege for file-system access |

**What they might ask:** "An attacker sends a crafted string `' OR 1=1 --` into a login form — what type of attack is this?" → **SQL injection.** "What is the best prevention?" → **Parameterized queries.**

**Exam tip:** Know that SQL injection targets the **database** through the **application**. The fix is in the application code (parameterized queries), but the DBA can also limit damage by enforcing least-privilege accounts for the application's DB connection.
