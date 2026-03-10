# Domain 4 — Data & Database Security: The Defense System

Domain 4 is not five separate objectives about encryption, compliance, access control, infrastructure, and attacks. It is one **defense system** — a set of concentric rings where every control exists because a specific threat exploits a specific vulnerability to compromise a specific property of data. Remove any ring and the threat it blocks reaches the next layer unchecked.

This is the second-highest-weighted domain (23%). The exam invests here because security is the domain where a single wrong answer in production has irreversible consequences — breaches cannot be un-breached, leaked data cannot be un-leaked.

---

## 1. The Defense System — One Model, Five Objectives

Security is not a list of controls. It's a system with three structural components: **what you're protecting** (data), **what you're protecting it from** (threats), and **how you prove you did it** (governance). The five objectives map directly to these components.

```
  ┌──────────────────────────────────────────────────────────────────────────┐
  │                                                                          │
  │    KNOW             PROTECT              DEFEND           PROVE          │
  │    ────             ───────              ──────           ─────          │
  │    Where is the     Control who can      Stop attacks     Demonstrate    │
  │    sensitive data?  see/touch it,        that bypass      to regulators  │
  │    What kind is it? and in what state    other controls   that controls  │
  │                                                          are in place   │
  │    ┌────────────┐   ┌────────────────┐   ┌────────────┐  ┌───────────┐  │
  │    │Data        │   │Encryption (4.1)│   │Infra       │  │Governance │  │
  │    │discovery   │──►│Masking    (4.1)│──►│security    │──►│& compliance│  │
  │    │& classifi- │   │Access     (4.3)│   │Physical/   │  │(4.2)      │  │
  │    │cation (4.2)│   │Destruction(4.1)│   │Logical(4.4)│  │           │  │
  │    │            │   │Audit      (4.1)│   │Attack      │  │DLP, reten-│  │
  │    │PII/PHI/PCI │   │IAM/Svc   (4.3)│   │defense(4.5)│  │tion, regs │  │
  │    └────────────┘   └────────────────┘   └────────────┘  └───────────┘  │
  │                                                                          │
  │    You can't        You can't protect    You can't defend  You can't     │
  │    protect what     data you don't       without infra     prove what    │
  │    you haven't      control access to    that enforces     you don't     │
  │    found                                 the controls      document      │
  │                                                                          │
  └──────────────────────────────────────────────────────────────────────────┘
```

**The dependency chain:** Know → Protect → Defend → Prove. Each phase depends on the one before it. Data discovery must happen before classification. Classification must happen before you can choose the right controls. Controls must be in place before infrastructure can enforce them. And all of it must be documented before you can prove compliance.

---

## 2. Governing Principles of Database Security

### Principle 1: Data Exists in States, and Every State Needs a Different Shield

Data is not a static thing. It moves through states, and each state exposes it to different threats:

```
  ┌─────────────────────────────────────────────────────────────────────────┐
  │                   THE DATA STATE MODEL                                  │
  │                                                                         │
  │  ┌───────────────┐    ┌───────────────┐    ┌───────────────┐           │
  │  │  IN TRANSIT    │    │   AT REST      │    │   IN USE       │           │
  │  │               │    │               │    │               │           │
  │  │ Moving between│    │ Stored on disk│    │ Being queried │           │
  │  │ client ↔ DB   │    │ DB files,     │    │ or displayed  │           │
  │  │ or DB ↔ DB    │    │ backups, logs │    │ to a user     │           │
  │  │               │    │               │    │               │           │
  │  │ Threat:       │    │ Threat:       │    │ Threat:       │           │
  │  │ Interception  │    │ Unauthorized  │    │ Unauthorized  │           │
  │  │ (on-path      │    │ file access,  │    │ viewing,      │           │
  │  │  attack)      │    │ theft of      │    │ over-          │           │
  │  │               │    │ media/backups │    │ privileged     │           │
  │  │ Shield:       │    │               │    │ access        │           │
  │  │ TLS/SSL       │    │ Shield:       │    │               │           │
  │  │ Client-side   │    │ TDE           │    │ Shield:       │           │
  │  │ encryption    │    │ Column-level  │    │ Views         │           │
  │  │               │    │ Full-disk     │    │ Dynamic       │           │
  │  │               │    │ (BitLocker/   │    │ masking       │           │
  │  │               │    │  LUKS)        │    │ Least         │           │
  │  │               │    │               │    │ privilege     │           │
  │  │               │    │               │    │ Row-level     │           │
  │  │               │    │               │    │ security      │           │
  │  └───────┬───────┘    └───────┬───────┘    └───────┬───────┘           │
  │          │                    │                    │                    │
  │          └────────────┬───────┘────────────────────┘                    │
  │                       ▼                                                 │
  │              ┌───────────────┐                                          │
  │              │  END OF LIFE   │                                          │
  │              │               │                                          │
  │              │ Data must be  │                                          │
  │              │ destroyed     │                                          │
  │              │               │                                          │
  │              │ Logical del   │  ← recoverable (soft delete)            │
  │              │ Physical del  │  ← may be forensically recoverable     │
  │              │ Crypto erasure│  ← destroy key = data gone forever     │
  │              │ Media sanit.  │  ← degauss, overwrite, shred           │
  │              └───────────────┘                                          │
  └─────────────────────────────────────────────────────────────────────────┘
```

**Exam implication:** When a question describes a data scenario, first determine the **state** — transit, rest, or use. The state determines which controls are relevant. "Protecting data as it travels to the database" ≠ "protecting data stored on disk."

### Principle 2: Authentication Answers "Who Are You?" — Authorization Answers "What Can You Do?"

These are sequential, dependent, and the exam tests the boundary between them:

```
  ┌──────────────────────────┐        ┌──────────────────────────┐
  │     AUTHENTICATION       │        │     AUTHORIZATION        │
  │     (Identity)           │        │     (Permission)         │
  │                          │        │                          │
  │  "Prove you are who      │───────►│  "Now that I know who    │
  │   you claim to be"       │        │   you are, here is what  │
  │                          │  then  │   you're allowed to do"  │
  │  Mechanisms:             │        │                          │
  │  • Password              │        │  Mechanisms:             │
  │  • MFA (something you    │        │  • GRANT / REVOKE (DCL)  │
  │    know + have + are)    │        │  • Role-based access     │
  │  • SSO                   │        │  • Least privilege       │
  │  • Biometrics            │        │  • Views (column-level   │
  │  • IAM / LDAP / AD      │        │    restriction)          │
  │                          │        │                          │
  │  Fails → no access at   │        │  Fails → access to      │
  │  all (door is locked)    │        │  wrong data (door is    │
  │                          │        │  open too wide)          │
  └──────────────────────────┘        └──────────────────────────┘

  Authentication without authorization = you're inside but can access everything.
  Authorization without authentication = anyone can claim any identity.
  You need BOTH. In this order.
```

### Principle 3: Every Attack Targets a Specific Layer

No attack is generic. Each one targets a specific point in the defense stack, and each has a specific primary mitigation:

```
  ATTACK                TARGET LAYER          PRIMARY MITIGATION
  ─────────────────────────────────────────────────────────────────

  SQL injection    ──►  Application / Data    Parameterized queries
  DoS              ──►  Network / Availability Rate limiting, firewalls
  On-path          ──►  Transport channel     TLS/SSL encryption
  Brute-force      ──►  Authentication        Lockout policies, MFA
  Phishing         ──►  Human (social eng.)   Training, MFA
  Ransomware       ──►  File system / Storage Offline/immutable backups
```

**The rule:** If the exam describes an attack and asks for the mitigation, identify the **layer** being targeted. The mitigation always matches the layer — you don't fix a transport-layer problem with a file-system solution.

### Principle 4: You Can't Protect What You Don't Know About

Data discovery is the prerequisite for everything else in Domain 4:

```
  Data discovery
       │
       ▼
  Data classification (PII? PHI? PCI?)
       │
       ▼
  Protection controls applied (encryption, masking, access control)
       │
       ▼
  Compliance documentation produced (prove controls match classification)
       │
       ▼
  Audit validates (external/internal auditor reviews evidence)
```

Skip data discovery → classification is incomplete → some sensitive data has no protection → compliance audit fails → breach exposes unprotected data → regulatory consequences.

### Principle 5: Physical Access Defeats All Logical Controls

If an attacker has physical access to the database server, every software-based control can be bypassed: they can boot from external media, copy disk files, or remove drives entirely. This is why the exam devotes an entire objective (4.4) to physical security that seems unrelated to databases.

```
  PHYSICAL SECURITY                           LOGICAL SECURITY
  (stops the body)                            (stops the packet)

  ┌────────────────────────────┐              ┌────────────────────────────┐
  │ If this layer fails...     │              │ If this layer fails...     │
  │                            │              │                            │
  │ All logical controls       │              │ Physical security still    │
  │ are meaningless.           │              │ prevents physical access.  │
  │                            │              │                            │
  │ Attacker can:              │              │ Attacker must:             │
  │ • Remove hard drives       │              │ • Penetrate remotely       │
  │ • Boot from USB            │              │ • Exploit software vulns   │
  │ • Access console           │              │ • Bypass authentication    │
  │ • Disable encryption       │              │                            │
  │   at the OS level          │              │ Still limited by:          │
  │                            │              │ • Encryption at rest       │
  │ Only defense remaining:    │              │ • Network segmentation     │
  │ • Full-disk encryption     │              │ • Least privilege          │
  │   (data still unreadable   │              │ • Audit logging            │
  │    without key)            │              │                            │
  └────────────────────────────┘              └────────────────────────────┘

  You need BOTH. They are not alternatives.
  Physical protects against on-site threats.
  Logical protects against remote threats.
```

### Principle 6: Defense in Depth Means No Single Point of Failure

The exam consistently favors answers that involve multiple layers over answers that rely on a single control. One layer failing must not mean total compromise:

```
  "How do we defend against SQL injection?"

  WRONG (single layer):              RIGHT (defense in depth):

  "Use parameterized queries"        Layer 1: Parameterized queries
                                              (prevents injection in app code)
  This IS the primary fix,                    +
  but the exam favors depth.         Layer 2: Least-privilege DB accounts
                                              (limits damage if injection succeeds)
                                              +
                                     Layer 3: WAF
                                              (blocks common injection patterns
                                               at the network perimeter)
                                              +
                                     Layer 4: Code auditing
                                              (finds injection vulnerabilities
                                               before attackers do)
                                              +
                                     Layer 5: Monitoring
                                              (detects anomalous queries that
                                               indicate injection attempts)
```

### Principle 7: Compliance Is the Proof Layer, Not the Protection Layer

Regulations (GDPR, PCI DSS, HIPAA) do not protect data. They define **requirements** that controls must meet and **evidence** you must produce. Compliance without actual security is a checkbox exercise. Security without compliance documentation means you can't prove your controls to anyone.

```
  SECURITY ≠ COMPLIANCE

  Security without compliance:          Compliance without security:
  "We encrypted everything              "We have all the paperwork
   and have perfect controls             but our actual controls
   but can't prove it to                 are weak and poorly
   the auditor"                          implemented"

  Result: audit fails                   Result: breach happens
  (controls exist but                   (documentation exists but
   evidence is missing)                  reality doesn't match)

  YOU NEED BOTH:
  Working controls (security) + documented evidence (compliance)
```

---

## 3. The Encryption Landscape

Encryption appears across multiple contexts in Domain 4. This map shows what each type protects, where it operates, and its key trade-off.

```
  ┌─────────────────────────────────────────────────────────────────────────┐
  │                    ENCRYPTION: THE COMPLETE MAP                         │
  │                                                                         │
  │  WHERE THE DATA IS        WHAT ENCRYPTS IT          TRADE-OFF           │
  │  ─────────────────        ──────────────────        ─────────           │
  │                                                                         │
  │  CLIENT ──► NETWORK       TLS/SSL                   Performance cost    │
  │  (in transit)             (in-transit encryption)    on every connection │
  │                           Encrypts the pipe.         Mitigates: on-path │
  │                           ┌─────────────────────┐    attacks             │
  │                           │ App ════════════ DB  │                       │
  │                           │     (encrypted)      │                       │
  │                           └─────────────────────┘                       │
  │                                                                         │
  │  CLIENT (before send)     Client-side encryption    App complexity;      │
  │  (in transit + at rest)   App encrypts data, then   server never sees   │
  │                           sends ciphertext.         plaintext.          │
  │                           Server stores it          Can't query          │
  │                           already encrypted.        encrypted values.   │
  │                                                                         │
  │  SERVER (after receive)   Server-side encryption    Provider manages    │
  │  (at rest)                Cloud provider encrypts   keys; less control  │
  │                           after data arrives.       for the customer.   │
  │                                                                         │
  │  DATABASE FILES           TDE (Transparent Data     Transparent to      │
  │  (at rest)                Encryption)               queries (decrypted  │
  │                           Whole-DB file encryption. in memory). Does    │
  │                           No app changes needed.    NOT protect data    │
  │                                                     in memory or in     │
  │                                                     transit.            │
  │                                                                         │
  │  SPECIFIC COLUMNS         Column-level encryption   Most granular.      │
  │  (at rest)                Only sensitive columns     Higher app          │
  │                           encrypted (SSN, card#).   complexity (must    │
  │                           Rest stays plaintext.     decrypt in code).   │
  │                                                                         │
  │  ENTIRE VOLUME            Full-disk encryption      Protects against    │
  │  (at rest)                BitLocker (Windows)       physical theft of   │
  │                           LUKS (Linux)              media. No DB-level  │
  │                           OS-level; DB unaware.     granularity.        │
  │                                                                         │
  └─────────────────────────────────────────────────────────────────────────┘
```

### Visual: Which Encryption Protects Against Which Threat

```
  THREAT                          ENCRYPTION THAT STOPS IT

  Attacker intercepts network  ──► TLS/SSL (in-transit)
  traffic between app and DB       Client-side encryption (data pre-encrypted)

  Attacker steals backup file  ──► TDE (file is ciphertext on disk)
  from disk or tape                Full-disk encryption
                                   Backup-level encryption (D5 crossover)

  Attacker gains DB login but  ──► Column-level encryption
  shouldn't see SSN column         (they can query, but SSN returns ciphertext)
                                   Dynamic masking (returns masked values)
                                   Views (column excluded entirely)

  Attacker physically steals   ──► Full-disk encryption
  the server / hard drive          (data unreadable without OS-level key)

  Data must be permanently     ──► Cryptographic erasure
  destroyed without physical       (destroy the key → data is gone)
  access to media
```

---

## 4. Data Masking — Protection for Data in Use

Masking and encryption solve different problems. Encryption makes data unreadable to everyone without a key. Masking makes data unreadable to specific people while keeping it usable for others.

```
  STATIC MASKING                              DYNAMIC MASKING
  ┌────────────────────────────────┐          ┌────────────────────────────────┐
  │                                │          │                                │
  │  WHEN: Before data leaves     │          │  WHEN: At query time,          │
  │  production (during copy)     │          │  in production                 │
  │                                │          │                                │
  │  WHAT CHANGES: The copy.      │          │  WHAT CHANGES: The result set. │
  │  Production data: untouched.  │          │  Production data: untouched.   │
  │                                │          │                                │
  │  ┌──────────┐  copy   ┌──────────┐       │  ┌──────────┐                  │
  │  │Production│ ──────► │Dev/Test  │       │  │Production│                  │
  │  │          │  +mask  │          │       │  │          │                  │
  │  │SSN: 123- │         │SSN: XXX- │       │  │SSN: 123- │                  │
  │  │45-6789   │         │XX-XXXX   │       │  │45-6789   │                  │
  │  └──────────┘         └──────────┘       │  └─────┬────┘                  │
  │                                │          │        │                        │
  │  USE CASE:                     │          │   query │                       │
  │  Realistic test data without   │          │        ▼                        │
  │  exposing real PII/PHI/PCI     │          │  ┌──────────┐ ┌──────────┐     │
  │                                │          │  │ DBA sees │ │ Analyst  │     │
  │  Masking is PERMANENT on       │          │  │ 123-45-  │ │ sees     │     │
  │  the copy. Irreversible.       │          │  │ 6789     │ │ XXX-XX-  │     │
  │                                │          │  │ (full    │ │ XXXX     │     │
  │                                │          │  │  access) │ │ (masked) │     │
  │                                │          │  └──────────┘ └──────────┘     │
  │                                │          │                                │
  │                                │          │  USE CASE:                     │
  │                                │          │  Role-based access in          │
  │                                │          │  production. Different users   │
  │                                │          │  see different views of the    │
  │                                │          │  same data.                    │
  └────────────────────────────────┘          └────────────────────────────────┘
```

### The Data Discovery Prerequisite

Before you can mask or classify anything, you must know where sensitive data lives:

```
  SCAN DATABASE(S)
       │
       ▼
  IDENTIFY columns containing:
  ┌─────────────────────────────────────────────────────────┐
  │  Pattern                    │  Likely classification    │
  │─────────────────────────────│───────────────────────────│
  │  SSN format (NNN-NN-NNNN)  │  PII                      │
  │  Email addresses           │  PII                      │
  │  Names + addresses + DOB   │  PII                      │
  │  Medical codes (ICD-10)    │  PHI                      │
  │  Diagnosis text fields     │  PHI                      │
  │  Insurance IDs             │  PHI                      │
  │  Card numbers (16 digits)  │  PCI DSS                  │
  │  CVV (3-4 digits)          │  PCI DSS                  │
  │  Cardholder name + expiry  │  PCI DSS                  │
  └─────────────────────────────────────────────────────────┘
       │
       ▼
  APPLY appropriate controls per classification
  (encryption level, masking rules, access restrictions,
   retention policy, audit requirements)
```

---

## 5. The Access Control System

Access control in Domain 4 is a layered system. Each layer narrows the scope of what a user can do:

```
  ┌──────────────────────────────────────────────────────────────────────┐
  │  LAYER 1: IDENTITY — Who are you?                                   │
  │                                                                      │
  │  IAM / Active Directory / LDAP → SSO → MFA                         │
  │  Centralized identity management. One identity, many systems.       │
  │  MFA adds: something you know + something you have + something     │
  │  you are (biometrics)                                               │
  ├──────────────────────────────────────────────────────────────────────┤
  │  LAYER 2: AUTHENTICATION — Prove it.                                │
  │                                                                      │
  │  Password policies enforce strength:                                │
  │  ┌──────────────────────────────────────────────────────────┐       │
  │  │ Complexity  │ Min length, mixed characters               │       │
  │  │ Expiration  │ Rotate every N days (privileged accounts)  │       │
  │  │ History     │ Can't reuse last N passwords               │       │
  │  │ Lockout     │ Lock after N failures → blocks brute-force│       │
  │  └──────────────────────────────────────────────────────────┘       │
  ├──────────────────────────────────────────────────────────────────────┤
  │  LAYER 3: AUTHORIZATION — What can you do?                          │
  │                                                                      │
  │  GRANT / REVOKE (DCL) → applies permissions to roles or users       │
  │                                                                      │
  │  Least privilege: default deny, then grant only what's needed       │
  │                                                                      │
  │  ┌────────────────────────────────────────────────┐                 │
  │  │  Need              Grant                      │                 │
  │  │  ─────────────     ────────────────────────    │                 │
  │  │  Read reports       SELECT on specific tables  │                 │
  │  │  Modify data        INSERT, UPDATE, DELETE     │                 │
  │  │  Run procedures     EXECUTE on specific procs  │                 │
  │  │  Manage schema      DDL permissions (rare)     │                 │
  │  │  Full admin         db_owner / sysadmin        │                 │
  │  │                     ↑ NEVER for routine tasks  │                 │
  │  └────────────────────────────────────────────────┘                 │
  ├──────────────────────────────────────────────────────────────────────┤
  │  LAYER 4: DATA-LEVEL RESTRICTIONS — What can you see?              │
  │                                                                      │
  │  Views          → Hide columns (SSN excluded from analyst view)    │
  │  Dynamic masking → Show masked values based on role                │
  │  Row-level       → Filter rows by user context                     │
  └──────────────────────────────────────────────────────────────────────┘
```

### Service Accounts — The Special Case

Service accounts are not user accounts. They exist for applications and automation, and have their own rules:

```
  USER ACCOUNT                             SERVICE ACCOUNT
  ┌──────────────────────────┐             ┌──────────────────────────┐
  │ Used by: a human         │             │ Used by: application     │
  │                          │             │ or automated process     │
  │ Authentication:          │             │                          │
  │ Interactive login        │             │ Authentication:          │
  │ (username + password     │             │ Non-interactive          │
  │  at a console or app)    │             │ (connection string,      │
  │                          │             │  secrets manager)        │
  │ NEVER use as a           │             │                          │
  │ service account          │             │ NEVER store password in: │
  │                          │             │ • Source code             │
  │                          │             │ • Config files (plaintext)│
  │                          │             │ • Shared documents        │
  │                          │             │                          │
  │                          │             │ DO:                      │
  │                          │             │ • Rotate on schedule     │
  │                          │             │ • Scope to minimum       │
  │                          │             │   permissions needed     │
  │                          │             │ • Use secrets manager    │
  │                          │             │ • Audit periodically     │
  └──────────────────────────┘             └──────────────────────────┘
```

---

## 6. The Threat Map — Attacks, Targets, and Mitigations

Every attack in 4.5 targets a specific layer of the defense system. Understanding the target layer tells you the mitigation — you don't fix a human-layer problem with a network-layer tool.

```
  ┌──────────────────────────────────────────────────────────────────────┐
  │                     THE ATTACK SURFACE                               │
  │                                                                      │
  │  LAYER              ATTACK              HOW IT WORKS                │
  │  ─────              ──────              ────────────                │
  │                                                                      │
  │  HUMAN         ───  Phishing       ───  Fake email/site tricks      │
  │  (social                                user into revealing         │
  │   engineering)                          credentials                  │
  │       │                                                              │
  │       ▼                                                              │
  │  AUTHENTICATION ──  Brute-force    ───  Automated repeated login    │
  │  (identity)                             attempts until one works    │
  │       │                                                              │
  │       ▼                                                              │
  │  NETWORK        ──  DoS            ───  Overwhelm server with       │
  │  (availability)                         traffic; legitimate users   │
  │       │                                 can't connect               │
  │       ▼                                                              │
  │  TRANSPORT      ──  On-path        ───  Intercept client ↔ server  │
  │  (channel)          (MitM)              communication; steal        │
  │       │                                 credentials or data         │
  │       ▼                                                              │
  │  APPLICATION    ──  SQL injection  ───  Malicious SQL via input     │
  │  (data)                                 fields; reads/modifies/     │
  │       │                                 deletes data                 │
  │       ▼                                                              │
  │  FILE SYSTEM    ──  Ransomware     ───  Encrypts DB files and       │
  │  (storage)                              demands payment for key     │
  │                                                                      │
  └──────────────────────────────────────────────────────────────────────┘
```

### Defense-in-Depth Mitigation Map

Each attack has a primary mitigation (the most effective single control) plus secondary layers:

```
  ATTACK          │ PRIMARY MITIGATION           │ DEFENSE-IN-DEPTH LAYERS
  ────────────────│──────────────────────────────│─────────────────────────────
                  │                              │
  SQL injection   │ Parameterized queries        │ + Least-privilege DB accounts
                  │ (prevents injection at       │ + WAF (blocks known patterns)
                  │  the code level)             │ + Code auditing (find vulns)
                  │                              │ + Input validation
                  │                              │ + Monitoring (anomalous queries)
                  │                              │
  DoS             │ Rate limiting + firewall     │ + Connection throttling
                  │ rules                        │ + Load balancing
                  │                              │ + Auto-scaling
                  │                              │ + Network segmentation
                  │                              │
  On-path (MitM)  │ TLS/SSL on all DB           │ + Certificate pinning
                  │ connections                  │ + Mutual authentication
                  │                              │ + Client-side encryption
                  │                              │
  Brute-force     │ Account lockout policies     │ + Strong password requirements
                  │                              │ + MFA
                  │                              │ + Monitoring failed logins
                  │                              │ + CAPTCHA / rate limiting
                  │                              │
  Phishing        │ Security awareness training  │ + MFA (stolen password alone
                  │                              │   isn't enough)
                  │                              │ + Email filtering
                  │                              │ + Verify sender identity
                  │                              │
  Ransomware      │ Offline / immutable backups  │ + Endpoint protection
                  │ (can restore without paying) │ + Network segmentation
                  │                              │ + Least privilege on file
                  │                              │   system (limits spread)
                  │                              │ + Full-disk encryption
                  │                              │   (attacker can't read even
                  │                              │    if they copy files)
```

### Visual: The SQL Injection Kill Chain

SQL injection is the most database-specific attack and the most tested. Here's how it works end-to-end:

```
  ATTACKER                    APPLICATION                   DATABASE
  ────────                    ───────────                   ────────

  Enters in login form:       Without parameterized         Receives and
  Username: admin             queries, app builds SQL       executes:
  Password: ' OR 1=1 --      by concatenation:
                                                            SELECT * FROM users
                              "SELECT * FROM users          WHERE username='admin'
                               WHERE username='" +          AND password=''
                               input + "' AND               OR 1=1 --'
                               password='" +
                               input + "'"                  1=1 is ALWAYS true
                                                            -- comments out the rest
                                                            → Returns ALL users
                              RESULT: attacker is           → Full database access
                              logged in as admin


  WITH PARAMETERIZED QUERIES (the fix):

  App builds:                 SELECT * FROM users
                              WHERE username = @user
                              AND password = @pass

                              @user = "admin"
                              @pass = "' OR 1=1 --"   ← treated as a LITERAL STRING
                                                         not as SQL code
                              → Login fails. Injection blocked.
```

---

## 7. Infrastructure Security — Two Dimensions of Defense

Physical and logical security are not two sections of a textbook. They are **two dimensions of a single defense perimeter**. Every threat arrives through one dimension or the other. A complete defense covers both.

```
  ┌──────────────────────────────────────────────────────────────────────┐
  │                   PHYSICAL SECURITY                                  │
  │                   (stops the body)                                   │
  │                                                                      │
  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌────────────┐ │
  │  │ ACCESS       │  │ SURVEILLANCE│  │ FIRE         │  │ COOLING    │ │
  │  │ CONTROL      │  │             │  │ SUPPRESSION  │  │ (HVAC)     │ │
  │  │              │  │ CCTV        │  │              │  │            │ │
  │  │ Locks        │  │ cameras     │  │ Clean-agent  │  │ Prevents   │ │
  │  │ Badge readers│  │ covering    │  │ (FM-200,     │  │ over-      │ │
  │  │ Mantraps     │  │ server room │  │  Novec)      │  │ heating    │ │
  │  │ Biometrics   │  │ and entry   │  │ Won't damage │  │ → hardware │ │
  │  │ (fingerprint,│  │ points      │  │ electronics  │  │   failure  │ │
  │  │  retinal)    │  │             │  │              │  │ → outages  │ │
  │  │              │  │ Recorded    │  │ Smoke detect │  │            │ │
  │  │ Prevents:    │  │ for forensic│  │ and alarms   │  │ Availability│ │
  │  │ unauthorized │  │ review      │  │              │  │ protection │ │
  │  │ physical     │  │             │  │ Availability │  │            │ │
  │  │ access       │  │ Deters and  │  │ + integrity  │  │            │ │
  │  │              │  │ documents   │  │ protection   │  │            │ │
  │  └─────────────┘  └─────────────┘  └─────────────┘  └────────────┘ │
  ├──────────────────────────────────────────────────────────────────────┤
  │                   LOGICAL SECURITY                                   │
  │                   (stops the packet)                                  │
  │                                                                      │
  │  ┌──────────────────┐  ┌──────────────────┐  ┌───────────────────┐  │
  │  │    FIREWALL       │  │ PERIMETER NETWORK │  │   PORT SECURITY   │  │
  │  │                  │  │      (DMZ)         │  │                   │  │
  │  │ Filter traffic   │  │                  │  │ Restrict open     │  │
  │  │ by IP, port,     │  │ Web servers HERE │  │ ports on DB       │  │
  │  │ protocol         │  │                  │  │ server            │  │
  │  │                  │  │ DB NEVER HERE     │  │                   │  │
  │  │ Allow only app   │  │                  │  │ Change defaults   │  │
  │  │ servers on DB    │  │ DB belongs on    │  │ where practical   │  │
  │  │ ports            │  │ internal network │  │                   │  │
  │  │                  │  │ behind firewall  │  │ Disable unused    │  │
  │  │ 1433 (SQL Svr)   │  │                  │  │ services          │  │
  │  │ 3306 (MySQL)     │  │                  │  │                   │  │
  │  │ 5432 (PostgreSQL)│  │                  │  │ Prevents:         │  │
  │  │ 27017 (MongoDB)  │  │                  │  │ reconnaissance    │  │
  │  └──────────────────┘  └──────────────────┘  │ and unauthorized  │  │
  │                                               │ services          │  │
  │                                               └───────────────────┘  │
  └──────────────────────────────────────────────────────────────────────┘
```

---

## 8. Governance — The Accountability System

Governance (4.2) is the layer that connects security controls to legal and regulatory requirements. It doesn't protect data directly — it ensures protection is **defined, measured, and provable**.

### The Classification → Regulation → Control Pipeline

```
  DATA CLASSIFICATION                REGULATION                  REQUIRED CONTROLS
  ──────────────────                 ──────────                  ─────────────────

  PII                          ───►  Various (CCPA,        ───►  Encryption
  (SSN, email, name,                 state/federal               Masking
   address, DOB)                     privacy laws)               Access restriction
                                                                 Breach notification
                                                                 Right to deletion

  PHI                          ───►  HIPAA                 ───►  Minimum necessary
  (medical records,                                              access
   diagnoses, insurance IDs)                                     Audit trails
                                                                 Safeguards
                                                                 Breach notification

  PCI DSS                     ───►  PCI SSC               ───►  Encryption of
  (card number, CVV,                                             cardholder data
   cardholder name, expiry)                                      Network segmentation
                                                                 Quarterly scans
                                                                 Access logging

  EU/EEA resident data        ───►  GDPR                  ───►  Right to erasure
                                                                 Data portability
                                                                 72-hour breach
                                                                 notification
                                                                 Lawful basis
                                                                 DPO required
```

**Exam decision rule:** If the question mentions the **data type**, it's pointing to the classification. If it mentions the **regulation name**, it's pointing to the required control. Match them:

| Data cue in question | Classification | Regulation |
|---|---|---|
| "Social Security number" | PII | Privacy laws |
| "medical diagnosis" | PHI | HIPAA |
| "credit card number" | PCI DSS | PCI SSC |
| "EU customer requests deletion" | GDPR | GDPR (right to erasure) |

### Data Retention and Destruction — The End-of-Life System

```
  DATA IS CREATED
       │
       ▼
  DATA IS ACTIVE (in use, queried, updated)
       │
       │  Retention policy defines how long to keep it
       │
       ▼
  RETENTION PERIOD EXPIRES
       │
       ├── Business still needs it?
       │     YES → ARCHIVE (move to low-cost, long-term storage)
       │
       └── No longer needed?
             │
             ▼
        DESTRUCTION (choose method based on sensitivity)
             │
             ├── Low sensitivity
             │     └── Logical deletion (soft delete; recoverable)
             │
             ├── Medium sensitivity
             │     └── Physical deletion (DELETE/TRUNCATE; may be
             │         forensically recoverable from backups/disk)
             │
             ├── High sensitivity
             │     └── Cryptographic erasure (destroy the encryption
             │         key; data is permanently unreadable)
             │
             └── Media being decommissioned
                   └── Media sanitization (degauss, overwrite,
                       or physically destroy the storage device)

  RISK:
  Over-retention → more data to protect → larger breach surface → higher cost
  Under-retention → compliance violation → legal consequences
```

### Data Loss Prevention (DLP)

DLP is the policy and tooling layer that prevents sensitive data from leaving the organization through unauthorized channels:

```
  DLP monitors and blocks:

  ┌───────────────────────────────────────────────────────────────┐
  │                                                               │
  │  Email with SSN in body or attachment   ──► BLOCKED           │
  │  Database export to unauthorized share  ──► BLOCKED           │
  │  USB copy of cardholder data            ──► BLOCKED           │
  │  Screenshot of PHI                      ──► LOGGED / ALERTED  │
  │  Clipboard containing PII              ──► BLOCKED           │
  │                                                               │
  │  DLP requires data classification to work.                    │
  │  Without knowing WHAT is sensitive, DLP can't detect it.      │
  └───────────────────────────────────────────────────────────────┘
```

---

## 9. The Security Audit — The Diagnostic Process

The security audit (4.1) ties Domain 4 back to Domain 3's operations loop. It's the mechanism for detecting security gaps before attackers exploit them.

```
  SECURITY AUDIT CHECKLIST (exam-tested sequence):

  ┌──────────────────────────────────────────────────────────────────┐
  │                                                                  │
  │  1. CHECK FOR EXPIRED / DORMANT ACCOUNTS                        │
  │     Attackers target accounts nobody is watching.               │
  │     Disable or remove accounts no longer in use.                │
  │                                                                  │
  │  2. REVIEW CONNECTION REQUEST LOGS                              │
  │     Look for: unauthorized IPs, unusual hours,                  │
  │     high volume of failed attempts (brute-force?),              │
  │     connections from unexpected geographies.                    │
  │                                                                  │
  │  3. AUDIT SQL CODE FOR INJECTION VULNERABILITIES                │
  │     Look for: dynamic SQL built by concatenation,               │
  │     missing parameterization, user input directly in queries.   │
  │                                                                  │
  │  4. VERIFY CREDENTIAL STORAGE                                   │
  │     No plaintext passwords anywhere.                            │
  │     No hardcoded connection strings in source code.             │
  │     Secrets manager in use.                                     │
  │                                                                  │
  │  5. CONFIRM PASSWORD POLICIES ARE ENFORCED                      │
  │     Complexity, expiration, history, lockout all configured.    │
  │                                                                  │
  │  6. VALIDATE LEAST PRIVILEGE ON ALL SERVICE ACCOUNTS            │
  │     No service account should have db_owner or sysadmin.        │
  │     Permissions scoped to specific application needs.           │
  │                                                                  │
  └──────────────────────────────────────────────────────────────────┘
```

---

## 10. Domain 4 Decision Engine

### Filter 1: Which security subsystem?

| Question is about... | Subsystem | Sub-objective |
|---|---|---|
| Encryption, masking, data destruction, discovery, audit | Data protection | 4.1 |
| DLP, retention, classification, PII/PHI/PCI, regulations | Governance | 4.2 |
| Passwords, service accounts, least privilege, IAM, MFA, SSO | Access control | 4.3 |
| Locks, cameras, fire, cooling, firewalls, DMZ, ports | Infrastructure | 4.4 |
| SQL injection, DoS, on-path, brute-force, phishing, ransomware | Attack defense | 4.5 |

### Filter 2: What data state?

| Data is described as... | State | Controls to consider |
|---|---|---|
| "traveling between," "sent to," "received by" | In transit | TLS/SSL, client-side encryption |
| "stored on disk," "backup file," "database file" | At rest | TDE, column-level, full-disk encryption |
| "displayed to user," "query returns," "role sees" | In use | Views, dynamic masking, least privilege |
| "no longer needed," "must be permanently removed" | End of life | Crypto erasure, media sanitization, logical/physical deletion |

### Filter 3: Is it a boundary question?

| Pair | Distinguishing property |
|---|---|
| Authentication vs Authorization | "Who are you?" vs "What can you do?" |
| Encryption vs Masking | Data unreadable to ALL without key vs data hidden from SOME users based on role |
| Static vs Dynamic masking | On a copy, permanent vs at query time, role-based |
| TDE vs Column-level encryption | Whole database file, transparent vs specific columns, app complexity |
| In-transit vs At-rest encryption | Protects the pipe vs protects the file |
| Client-side vs Server-side encryption | App encrypts before sending vs server encrypts after receiving |
| Logical vs Physical deletion | Soft delete (recoverable) vs DELETE/TRUNCATE (may be forensically recoverable) |
| Cryptographic erasure vs Media sanitization | Destroy the key (logical) vs destroy the media (physical) |
| PII vs PHI vs PCI DSS | General personal data vs health data (HIPAA) vs payment card data (PCI SSC) |
| GDPR vs Regional regulations | EU/EEA global (right to erasure, 72h breach) vs jurisdiction-specific |
| Physical vs Logical security | Stops the body (locks, cameras) vs stops the packet (firewalls, ports) |
| Firewall vs WAF | Network-layer filtering (IP/port) vs application-layer filtering (HTTP/SQL patterns) |
| DLP vs Encryption | Prevents data from LEAVING vs makes data UNREADABLE |

### Filter 4: Is it an attack identification question?

| Scenario description | Attack type | Primary fix |
|---|---|---|
| "Malicious string `' OR 1=1 --` in a login form" | SQL injection | Parameterized queries |
| "Server overwhelmed, legitimate users can't connect" | DoS | Rate limiting, firewalls |
| "Attacker intercepts communication between app and DB" | On-path (MitM) | TLS/SSL |
| "Automated repeated login attempts from single IP" | Brute-force | Lockout + MFA |
| "Fake email tricks DBA into revealing password" | Phishing | Training + MFA |
| "DB files encrypted, ransom demanded for key" | Ransomware | Offline/immutable backups |

### Filter 5: Is it a "what should the DBA do?" scenario question?

| Scenario | Action | Why |
|---|---|---|
| New analyst needs read-only access | Create/assign read-only role with SELECT only | Least privilege |
| Service account password hasn't changed in 2 years | Rotate immediately, enforce rotation policy | Credential hygiene |
| Developer asks for db_owner to debug | Deny; grant only specific permissions needed | Never grant admin for routine tasks |
| Auditor requests PCI DSS proof | Provide compliance docs, data dictionary showing encryption, audit log review | Prove controls are in place |
| User requests deletion of personal data | Evaluate right to erasure (GDPR) | Regulation requires it |
| Database refresh to dev environment | Mask sensitive data before copying | PII/PHI in dev = compliance violation |

---

## 11. Domain 4 Ripple Effects

| If this fails... | Cascade through the system... |
|---|---|
| **Data discovery skipped** | → Sensitive data locations unknown → Classification incomplete → Some PII/PHI has no encryption or masking → Database refresh copies unmasked data to dev → Developer exposes it → Breach notification required → Regulatory investigation |
| **Parameterized queries not used** | → SQL injection vulnerability exists → Attacker discovers it → Unauthorized data access → Data exfiltration → Breach of PII/PHI/PCI → Compliance violation → Legal and financial consequences → Loss of customer trust |
| **No TLS on DB connections** | → On-path attack intercepts credentials → Attacker authenticates as legitimate user → Has whatever privileges that user has → If service account with broad permissions → Full database access → Data breach |
| **Service account has db_owner** | → Application compromise = full DB compromise → Attacker can: read all data, modify schema, drop tables, create new accounts → Blast radius is maximum → Every table, every row exposed |
| **No MFA configured** | → Phished credentials alone grant access → Brute-forced password alone grants access → Single factor = single point of failure → Attacker authenticated as DBA → Privilege escalation to any action |
| **Backups not offline/immutable** | → Ransomware encrypts live DB AND backup files → No recovery without paying ransom → Even paying doesn't guarantee decryption → Extended downtime → Data potentially lost permanently |
| **No account lockout policy** | → Brute-force attack runs indefinitely → Eventually cracks weak passwords → Unauthorized access → No alert fires because no lockout threshold exists → Attack succeeds silently |
| **Password stored in source code** | → Source code committed to repo → Anyone with repo access has DB credentials → Credentials may leak to public (GitHub exposure) → Attacker uses credentials → Authenticated access without any "attack" needed |
| **DB placed in DMZ** | → Database directly exposed to internet → Every internet-connected attacker can attempt connections → Attack surface is maximum → Firewall is the ONLY defense (no depth) → Single firewall misconfiguration = full exposure |
| **Audit logs not reviewed** | → Privilege escalation goes undetected → Unauthorized schema changes happen silently → Data exfiltration occurs over weeks → Compliance auditor asks for evidence → None available → Audit failure + undiscovered breach |
