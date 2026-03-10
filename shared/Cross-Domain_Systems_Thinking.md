# CompTIA DataSys+ DS0-001 — Systems Thinking Guide

The five exam domains are not five separate topics. They are five views of **one system**: a database that must be designed, built, operated, protected, and survived. Every concept exists because of a consequence somewhere else in the system.

This guide maps the deep structure — the "why behind the what" — so you can derive answers from principles instead of memorizing isolated facts.

---

## 1. The Single System: One Lifecycle, Five Lenses

Every database follows the same lifecycle. The five domains are just different phases of that lifecycle, and the exam tests whether you understand which phase a question lives in.

```
DESIGN (Domain 1) → BUILD (Domain 2) → OPERATE (Domain 3) → PROTECT (Domain 4) → SURVIVE (Domain 5)
    ↑                                                                                       |
    └───────────────────────── lessons learned feed back ──────────────────────────────────────┘
```

| Phase | Domain | Core Question | What You're Doing |
|---|---|---|---|
| **DESIGN** | 1 – Fundamentals | "What kind of database, and how should it be structured?" | Choosing relational vs NoSQL, writing SQL, designing schemas, planning requirements |
| **BUILD** | 2 – Deployment | "How do we get it running correctly?" | Installing, configuring, connecting, importing data, testing everything |
| **OPERATE** | 3 – Management | "How do we keep it healthy and evolving?" | Monitoring, optimizing, patching, documenting, managing changes |
| **PROTECT** | 4 – Security | "How do we prevent unauthorized access, leaks, and attacks?" | Encrypting, controlling access, classifying data, defending against threats |
| **SURVIVE** | 5 – Continuity | "How do we recover when things go wrong?" | Backing up, replicating, planning for disaster, failing over and back |

**The key insight:** A concept that appears in multiple domains (like "indexes") isn't being repeated — it's being viewed from a different lifecycle phase. The exam tests whether you can shift lenses.

---

## 2. Governing Principles

These ~10 rules explain the logic behind 80% of exam answers. When you're unsure, apply the relevant principle.

### Principle 1: Data Has Gravity

The more sensitive or voluminous data becomes, the more infrastructure it attracts: encryption, masking, backups, monitoring, compliance documentation, access controls. Every security and continuity mechanism exists because data is valuable and vulnerable.

```
                          PUBLIC data
                       (employee count)
                     ┌───────────────────┐
                     │  Schema design    │
                     │  Basic backups    │
                     └───────┬───────────┘
                             │ add sensitivity...
                     ┌───────▼───────────────────────┐
                     │     INTERNAL data              │
                     │   (revenue, contracts)          │
                     │  + Access control               │
                     │  + Monitoring                   │
                     │  + Change management             │
                     └───────┬───────────────────────┘
                             │ add sensitivity...
                ┌────────────▼──────────────────────────────┐
                │         CONFIDENTIAL data                  │
                │       (PII, PHI, PCI DSS)                  │
                │  + Encryption (transit + rest)              │
                │  + Masking (static + dynamic)               │
                │  + Data classification & discovery          │
                │  + Compliance documentation                 │
                │  + Audit logging & periodic review          │
                │  + Retention policies & secure destruction  │
                │  + Incident response procedures             │
                └──────────────────────────────────────────┘
```

Each ring adds controls. The exam expects you to recognize that sensitive data demands **all** the outer rings, not just one.

**Implication:** When a question describes sensitive data (PII, PHI, PCI), expect the answer to involve multiple layers — not just one control.

### Principle 2: Every Layer Exists Because the Layer Below It Can Fail

| Layer | Protects Against | If It Fails... |
|---|---|---|
| Backups (D5) | Data loss | You lose data permanently |
| Monitoring (D3) | Silent degradation | Problems compound until outage |
| Encryption (D4) | Unauthorized reading | Data breach |
| Access control (D4) | Unauthorized action | Privilege escalation, data modification |
| Referential integrity (D1) | Orphan/corrupt data | Application errors, wrong query results |
| Schema design (D1) | Structural mismatch | Performance problems, inability to query |

**Implication:** "What should the DBA do first?" questions are asking you to identify which layer is most foundational to the situation described.

### Principle 3: Structure Before Data, Data Before Users

The lifecycle has a strict dependency order. You cannot skip steps.

```
Schema exists → Data loaded → Indexes built → Users connected → Queries optimized → Changes managed
```

If a question describes a problem, trace backward along this chain to find the root cause.

### Principle 4: The Closer to Production, the More Process Required

| Environment | Process Required |
|---|---|
| Dev/test | Minimal — experiment freely |
| Staging | Some — regression testing |
| Production | Maximum — change management, CAB approval, communication, validation, documentation |

```
                 LOW PROCESS                              HIGH PROCESS
  ┌──────────────┐     ┌──────────────┐     ┌──────────────────────────────┐
  │     DEV      │     │   STAGING    │     │         PRODUCTION           │
  │              │     │              │     │                              │
  │ Experiment   │────►│ Regression   │────►│ Change management required   │
  │ freely       │     │ testing      │     │                              │
  │              │     │              │     │ Plan ──► Approve (CAB)       │
  │ DHCP OK      │     │ Validate     │     │   ──► Communicate            │
  │ Shared creds │     │ patches here │     │   ──► Implement              │
  │ Sample data  │     │ first        │     │   ──► Validate               │
  │              │     │              │     │   ──► Document                │
  │              │     │              │     │                              │
  │              │     │              │     │ Static IP · Service accounts │
  │              │     │              │     │ Full monitoring · Backups    │
  └──────────────┘     └──────────────┘     └──────────────────────────────┘
        │                     │                          │
        │                     │                          │
   Masked data ◄──── Database refresh ◄──── Production data
   (no real PII)      (mask BEFORE        (real sensitive data)
                       copying down)
```

Data flows **down** (production → staging → dev) via database refresh, but must be masked first. Process requirements flow **up** — the closer to production, the more gates.

**Implication:** Any question mentioning "production" should trigger you to think about change management (plan → approve → communicate → implement → validate → document).

### Principle 5: Prevention Is Cheaper Than Recovery

The exam consistently favors proactive answers over reactive ones.

| Proactive (preferred) | Reactive (last resort) |
|---|---|
| Parameterized queries → prevent SQL injection | Incident response after breach |
| Baseline + monitoring → catch degradation early | Emergency troubleshooting during outage |
| Test backups regularly → confirm they work | Discover backup is corrupt during DR event |
| Least privilege → limit blast radius | Revoke privileges after damage is done |
| Patch promptly → close vulnerabilities | Remediate after exploitation |

### Principle 6: Least Privilege Is the Universal Security Answer

When a security question doesn't clearly point to a specific control, "apply least privilege" is almost always correct or part of the correct answer. It applies to:
- User accounts (grant only needed permissions)
- Service accounts (scope to specific application needs)
- Application DB connections (never use admin credentials)
- File system access (limits ransomware blast radius)
- Network access (only required ports open)

### Principle 7: Synchronous = Safer but Slower

This trade-off appears in multiple forms across the exam:

| Mechanism | Synchronous | Asynchronous |
|---|---|---|
| Replication | Near-zero RPO, adds write latency | Faster writes, risk of recent data loss |
| Mirroring | High safety mode | High performance mode |
| Commit behavior | Transaction waits for confirmation | Transaction completes immediately |

**The rule:** If the question prioritizes **data safety**, choose synchronous. If it prioritizes **performance**, choose asynchronous.

### Principle 8: The Exam Tests Boundaries, Not Centers

Most questions target the **edges** between categories — the places where two concepts are easy to confuse:

- TRUNCATE (DDL) vs DELETE (DML)
- Differential (since last full) vs Incremental (since last backup)
- View (always current) vs Materialized view (stale until refreshed)
- RPO (data loss tolerance) vs RTO (downtime tolerance)
- Static masking (copy) vs Dynamic masking (query time)
- Trigger (automatic) vs Stored procedure (called)

**Implication:** For every concept, know what it is NOT. The wrong answer is usually the concept on the other side of the boundary.

### Principle 9: Every Monitoring Metric Is Meaningless Without a Baseline

The exam tests this directly: "CPU is at 80% — is this a problem?" The answer is always "depends on the baseline." This principle applies to every metric: throughput, disk usage, replication lag, query duration.

Without a baseline → no anomaly detection → no proactive alerting → silent failure.

### Principle 10: Documentation Is Not Optional — It's Operational Infrastructure

The exam treats documentation as first-class operational output, not paperwork:

| Document | Operational Function |
|---|---|
| Data dictionary | Single source of truth for schema — referenced during builds, changes, audits |
| ERD | Validates design before build; onboards new team members; satisfies audit |
| SOPs | Ensures consistency across DBAs; required for compliance |
| Maintenance docs | Runbooks for backups, index rebuilds, patching — operational continuity |
| Compliance docs | Evidence for auditors (PCI DSS, GDPR, SOC 2) |
| DR documentation | Updated after every DR event with lessons learned |

---

## 3. Causal Chains — Tracing Concepts Through the System

Each row below traces a single concept across its full lifecycle. This is how the exam creates cross-domain questions.

### The Index Chain

```
DESIGN (D1)          BUILD (D2)              OPERATE (D3)                    PROTECT (D4)
CREATE INDEX →        Index analysis          Rebuild if fragmented (>30%)    Indexes don't affect
(SQL DDL)             during validation →     Reorganize if light (5-30%) →  security directly, but
                                              Remove unused indexes →         covering indexes reduce
                                              Add covering indexes            data exposure in queries
```

**Exam pattern:** A question about index *syntax* is D1. About index *validation after import* is D2. About index *fragmentation* is D3.

### The Encryption Chain

```
PROTECT (D4)                                  SURVIVE (D5)
Client-side / In-transit / Server-side →      Backup files must ALSO be encrypted →
TDE / Column-level / Full-disk →              Encryption keys must be retained for
                                              the SAME duration as the backup →
                                              Cryptographic erasure = destroy key = data gone
```

**The rule:** Encryption at rest protects live data. Encryption of backups protects dead data. Lose the key to either, and the data is gone — intentionally (erasure) or accidentally (disaster).

### The Backup Chain

```
OPERATE (D3)              SURVIVE (D5)                              PROTECT (D4)
Monitor backup alerts →   Schedule: full → diff/incr → log →       Encrypt backup files →
Alert on failure →        Test restores (hope ≠ backup) →           Retain keys as long
Validate file size →      Validate hash before restore →            as backups →
                          Store: on-site + off-site (3-2-1) →       Mask data before
                          Purge or archive per retention policy      refresh to lower envs
```

**The rule:** A backup you've never tested is not a backup. A backup without its encryption key is not a backup. A backup on-site only is not survivable.

### Visual: Backup Types Over a Week

The critical distinction: differential captures changes since **last full**. Incremental captures changes since **last backup of any type**.

```
DIFFERENTIAL STRATEGY
                                                                 To restore
  Sun        Mon        Tue        Wed        Thu        Fri     Wednesday:
  ┌────┐     ┌────┐     ┌────┐     ┌────┐     ┌────┐     ┌────┐
  │FULL│     │Diff│     │Diff│     │Diff│     │Diff│     │Diff│  Need: FULL
  │████│     │░░  │     │░░░ │     │░░░░│     │░░░░░│    │░░░░░░│        +
  │████│     │    │     │    │     │    │     │     │    │      │  latest Diff
  └────┘     └────┘     └────┘     └────┘     └────┘     └────┘
  ALL data   Changes    Changes    Changes    Changes    Changes  = 2 files
             since Sun  since Sun  since Sun  since Sun  since Sun
                        ◄── each diff GROWS (captures more) ──►

INCREMENTAL STRATEGY
                                                                 To restore
  Sun        Mon        Tue        Wed        Thu        Fri     Wednesday:
  ┌────┐     ┌────┐     ┌────┐     ┌────┐     ┌────┐     ┌────┐
  │FULL│     │Inc │     │Inc │     │Inc │     │Inc │     │Inc │  Need: FULL
  │████│     │░   │     │░   │     │░   │     │░   │     │░   │        +
  │████│     │    │     │    │     │    │     │    │     │    │  Mon Inc
  └────┘     └────┘     └────┘     └────┘     └────┘     └────┘  Tue Inc
  ALL data   Changes    Changes    Changes    Changes    Changes  Wed Inc
             since Sun  since Mon  since Tue  since Wed  since Thu
             ◄── each inc stays SMALL (only since last) ──►       = 4 files

THE 3-2-1 RULE
  ┌─────────────────────────────────────────────────────────────┐
  │  3 copies        2 media types         1 off-site          │
  │                                                             │
  │  ┌────┐          ┌────┐  ┌────┐        ┌────┐              │
  │  │ DB │ (live)   │Disk│  │Tape│        │Cloud│ or          │
  │  ├────┤          └────┘  └────┘        │vault│ remote      │
  │  │Copy│ (local)                        └────┘  site        │
  │  ├────┤                                                     │
  │  │Copy│ (remote)  Protects against      Protects against    │
  │  └────┘           single media          site-level          │
  │                   failure               disaster            │
  └─────────────────────────────────────────────────────────────┘
```

### The Access Control Chain

```
DESIGN (D1)                BUILD (D2)              PROTECT (D4)              OPERATE (D3)
DCL: GRANT / REVOKE →     Provision service        Least privilege →         Periodic review →
Understand permissions     accounts before          Password policies →       Remove expired accounts →
(SELECT, INSERT, etc.)     deployment →             IAM / SSO / MFA →        Rotate service account
                                                    Role-based access         passwords → Audit logs
```

### The Schema Chain

```
DESIGN (D1)                BUILD (D2)                    OPERATE (D3)
Logical schema             Build physical schema →       Update data dictionary when
(platform-independent) →   from logical →                schema changes →
                           Create view schema →          Keep ERD current for audits →
                           Validate schema meets         Change management for any
                           requirements during testing   schema modifications
```

### Visual: Schema Levels — How They Transform

The three schema levels aren't alternatives — they're a transformation pipeline. Each level adds platform specificity.

```
  LOGICAL SCHEMA                 PHYSICAL SCHEMA               VIEW SCHEMA
  (Domain 1 - DESIGN)            (Domain 2 - BUILD)            (Domain 2/3)
  Platform-independent           Platform-specific              User-facing

  ┌─────────────────────┐        ┌─────────────────────┐       ┌──────────────────┐
  │ Customer             │        │ dbo.Customer         │       │ vw_CustomerList   │
  │   - CustomerID (PK) │───────►│   CustomerID INT PK  │──────►│   Name            │
  │   - Name            │        │   Name NVARCHAR(100) │       │   Email           │
  │   - Email           │ build  │   Email VARCHAR(255) │ expose│                    │
  │   - SSN             │        │   SSN CHAR(9)        │       │   (SSN hidden —   │
  │                     │        │                      │       │    confidentiality)│
  │ Order               │        │   INDEX ix_email     │       └──────────────────┘
  │   - OrderID (PK)    │        │   TABLESPACE users   │
  │   - CustomerID (FK) │        │                      │       ┌──────────────────┐
  │   - Amount          │        │ dbo.Order            │       │ vw_OrderSummary   │
  │                     │        │   OrderID INT PK     │──────►│   CustomerName    │
  │ Relationship:       │        │   CustomerID INT FK  │       │   TotalOrders     │
  │   Customer 1:M Order│        │   Amount DECIMAL     │       │   TotalAmount     │
  └─────────────────────┘        │   ON DELETE CASCADE  │       │                    │
                                 │                      │       │ (materialized?     │
  Contains:                      │   PARTITION BY year  │       │  if expensive      │
  • Entities                     │   ENGINE = InnoDB    │       │  aggregation)      │
  • Attributes                   └─────────────────────┘       └──────────────────┘
  • Relationships
  • Constraints                  Contains:                      Contains:
                                 • Tables, data types           • Virtual tables
  Validated via ERD              • Indexes, partitions          • Column subsets
                                 • Storage engine, tablespace   • Aggregations
                                 • Platform-specific syntax     • Security boundary
```

---

## 4. The Protection Stack — Why Everything Exists

Every mechanism in the exam exists to protect one of three properties. This is the CIA triad applied to databases:

```
┌──────────────────────────────────────────────────────────────────┐
│                     CONFIDENTIALITY                             │
│     Only authorized people see data                             │
│     Encryption (D4) · Access control (D4) · Masking (D4)       │
│     Views to restrict columns (D1) · Firewall/DMZ (D4)         │
├──────────────────────────────────────────────────────────────────┤
│                       INTEGRITY                                 │
│     Data is accurate and uncorrupted                            │
│     ACID transactions (D1) · Referential integrity (D1)        │
│     Consistency checks / DBCC CHECKDB (D3) · Hash validation   │
│     (D5) · Negative testing (D2) · Parameterized queries (D4)  │
├──────────────────────────────────────────────────────────────────┤
│                      AVAILABILITY                               │
│     Data is accessible when needed                              │
│     Monitoring + baselines (D3) · HA clustering (D5)           │
│     Replication + mirroring (D5) · Backups (D5)                │
│     Load balancing (D3) · Patch management (D3)                │
│     DoS mitigation (D4) · Capacity planning (D3)              │
└──────────────────────────────────────────────────────────────────┘
```

**When stuck on a question:** Ask "Is this about confidentiality, integrity, or availability?" The answer narrows your options immediately.

### Where Controls Physically Live — The Three-Tier View

Every security and operational control has a physical home. The exam tests whether you know which tier owns which responsibility.

```
INTERNET
    │
    │  ┌─────────────────────────────────────────────────────────┐
    │  │  PERIMETER (DMZ)                                        │
    │  │  ┌───────────────────────────────────────────────────┐  │
    │  │  │ WAF · Firewall · Load balancer · Web servers      │  │
    │  │  │                                                   │  │
    │  │  │ DB NEVER here                                     │  │
    │  │  └───────────────────────┬───────────────────────────┘  │
    │  │                          │ only allowed ports            │
    │  │  ┌───────────────────────▼───────────────────────────┐  │
    │  │  │ APPLICATION TIER                                  │  │
    │  │  │ App servers · ORMs · Connection pools              │  │
    │  │  │ Parameterized queries · Service accounts           │  │
    │  │  │                                                   │  │
    │  │  │ Controls: input validation, least-privilege        │  │
    │  │  │ connections, credential management (secrets mgr)   │  │
    │  │  └───────────────────────┬───────────────────────────┘  │
    │  │                          │ DB ports only (1433/3306/    │
    │  │                          │ 5432/27017)                  │
    │  │  ┌───────────────────────▼───────────────────────────┐  │
    │  │  │ DATABASE TIER                                     │  │
    │  │  │ DBMS · Storage · Replication · Backups             │  │
    │  │  │                                                   │  │
    │  │  │ Controls: TDE/column encryption · Access control   │  │
    │  │  │ Audit logging · Monitoring · ACID enforcement      │  │
    │  │  │ Static IP · Firewall rules · No internet exposure  │  │
    │  │  └───────────────────────────────────────────────────┘  │
    │  │                                                         │
    │  │  INTERNAL NETWORK (VLAN / VPC)                          │
    │  └─────────────────────────────────────────────────────────┘
```

An attack must penetrate each layer. Defense in depth means no single layer's failure is catastrophic.

### Defense in Depth — Layered Security at the Database Tier

Zooming into the database tier, security wraps the data in concentric rings:

```
┌─────────────────────────────────────────────────────────────┐
│  NETWORK LAYER                                              │
│  Firewall rules · Port restrictions · VLAN/VPC isolation    │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  HOST LAYER                                           │  │
│  │  OS hardening · Full-disk encryption (BitLocker/LUKS) │  │
│  │  Patch management · Disable unused services           │  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │  DATABASE LAYER                                 │  │  │
│  │  │  TDE · Least privilege · Audit logs · DBCC      │  │  │
│  │  │  ┌───────────────────────────────────────────┐  │  │  │
│  │  │  │  TABLE / COLUMN LAYER                     │  │  │  │
│  │  │  │  Column-level encryption · Views          │  │  │  │
│  │  │  │  Data masking (static / dynamic)          │  │  │  │
│  │  │  │  ┌─────────────────────────────────────┐  │  │  │  │
│  │  │  │  │  ROW LAYER                          │  │  │  │  │
│  │  │  │  │  Row-level security · Triggers       │  │  │  │  │
│  │  │  │  │  Parameterized queries (prevent      │  │  │  │  │
│  │  │  │  │  injection at the value level)       │  │  │  │  │
│  │  │  │  └─────────────────────────────────────┘  │  │  │  │
│  │  │  └───────────────────────────────────────────┘  │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## 5. Pattern Recognition — Repeating Structures

The exam reuses the same structural patterns across domains. Recognizing the pattern lets you predict the answer format.

### Pattern: The Spectrum

Many concepts sit on a spectrum from "more control / more work" to "less control / less work":

```
IaaS ←————————————→ PaaS ←————————————→ SaaS
(you manage most)    (shared)            (provider manages all)

Synchronous ←————————————————————→ Asynchronous
(safer, slower)                    (faster, riskier)

Full backup ←———→ Differential ←———→ Incremental
(biggest, simplest restore)        (smallest, most complex restore)

On-premises ←————————————————————→ Cloud
(full control, CapEx)              (elastic, OpEx)

Static masking ←————————————————→ Dynamic masking
(permanent, on copy)               (at query time, role-based)
```

**The rule:** The exam often asks you to place a scenario on one of these spectrums. The extremes have clear trade-offs — the answer depends on what the scenario prioritizes.

### Visual: RPO and RTO on a Disaster Timeline

RPO looks backward (how much data can we lose). RTO looks forward (how fast must we recover). They drive completely different decisions.

```
                           DISASTER
                           STRIKES
                              │
  ◄──── RPO ────►             │            ◄───── RTO ─────►
  (data loss window)          │            (downtime window)
                              │
──────────────────────────────┼──────────────────────────────────► time
                              │
  Last good           Data created         System         Users
  backup              since last backup    restored       reconnected
  exists              IS LOST              from backup    and working
  here                                     or replica
                              │
                              │
  RPO drives:                 │            RTO drives:
  • Backup frequency          │            • Architecture (HA, hot standby)
  • Replication type          │            • DR technique choice
    (sync = near-zero RPO)    │            • Budget (lower RTO = higher cost)
  • Log shipping interval     │            • Failover automation
```

```
  RPO/RTO vs DR Technique:

  COST & COMPLEXITY ──────────────────────────────────► HIGH
  RPO  ──────────────────────────────────────────────► NEAR-ZERO
  RTO  ──────────────────────────────────────────────► NEAR-ZERO

  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
  │ Log shipping │  │ Async        │  │ Sync         │  │ HA           │
  │              │  │ replication  │  │ replication  │  │ clustering   │
  │ RPO: hours   │  │ RPO: seconds │  │ RPO: ~zero   │  │ RPO: ~zero   │
  │ RTO: hours   │  │ RTO: minutes │  │ RTO: seconds │  │ RTO: ~zero   │
  │              │  │              │  │              │  │              │
  │ Cheapest     │  │ Good balance │  │ Write        │  │ Most complex │
  │ Most loss    │  │              │  │ latency cost │  │ Most $ $ $   │
  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘
```

### Pattern: The Ordered Process

The exam loves sequence questions ("what first?"). Every ordered process in the exam follows the same meta-pattern:

```
ASSESS → PLAN → APPROVE → EXECUTE → VALIDATE → DOCUMENT
```

This maps directly to:
- **Change management:** Plan → Approve → Communicate → Implement → Validate → Document
- **Deployment:** Assets → Install → Provision → Import → Test/Validate
- **ORM impact:** Review SQL → Validate → Assess impact → Provide solutions
- **Failback:** Verify primary → Resync → Validate integrity → Switch traffic → Monitor → Document
- **Security audit:** Check accounts → Review logs → Audit code → Verify credentials → Confirm policies → Validate privileges

**The rule:** The first step is always **assessment/review**, never implementation. The last step is always **documentation/monitoring**.

### Pattern: The "Never" Rules

Absolute rules that the exam tests as traps — if you see a question that violates one, it's the wrong answer:

| Never | Why |
|---|---|
| DB **never** in the DMZ | Physical proximity to the internet = maximum exposure |
| **Never** grant sysadmin/db_owner for routine tasks | Violates least privilege |
| **Never** store passwords in plaintext or source code | Credential exposure |
| **Never** skip testing patches in non-production first | Risk of breaking production |
| **Never** assume a backup works without testing a restore | Hope is not a strategy |
| **Never** apply changes to production without approval | Change management violation |
| **Never** use interactive credentials as service accounts | Security and audit risk |

### Pattern: The "First Step" Rules

When the question asks "what should the DBA do first?":

| Scenario | First Step | Why |
|---|---|---|
| ORM performance degradation | Review the generated SQL | Can't fix what you can't see |
| Unknown CPU spike | Compare to the baseline | 80% CPU may be normal |
| Post-import data issues | Validate data values/mapping | Root cause is in the source data |
| Security incident | Check for expired/dormant accounts | Lowest-hanging vulnerability |
| Planning a new database | Requirements gathering | Can't design without requirements |
| Production change requested | Plan (then get CAB approval) | Process before action |

---

## 6. The Dependency Graph — What Breaks What

Understanding dependencies tells you why the exam cares about each topic. If A depends on B, a question about A failing often has B as the root cause.

### Upward Dependencies (if the bottom breaks, everything above it breaks)

```
    Query performance
         ↑ depends on
    Index optimization
         ↑ depends on
    Physical schema design
         ↑ depends on
    Logical schema design
         ↑ depends on
    Requirements gathering
```

### The Backup Dependency Tree

```
    Successful DR failover
         ↑ depends on
    Tested, valid backups
         ↑ depends on
    Backup hash validation + restore testing
         ↑ depends on
    Scheduled, automated backups
         ↑ depends on
    RPO definition (how much data loss is acceptable)
         ↑ depends on
    Business requirements
```

If any link in this chain breaks, everything above it is unreliable.

### The Security Dependency Tree

```
    Regulatory compliance (GDPR, PCI DSS, HIPAA)
         ↑ depends on
    Audit logs + compliance documentation
         ↑ depends on
    Data classification (PII, PHI, PCI)
         ↑ depends on
    Data discovery (you can't protect what you don't know about)
```

```
    Defense against SQL injection
         ↑ depends on
    Parameterized queries (application layer)
         +
    Least-privilege DB accounts (database layer)
         +
    WAF (network layer)
```

Defense in depth: multiple layers, because any single layer can fail.

---

## 7. The Unified Decision Engine

When reading an exam question, run it through these filters in order:

### Filter 1: Which lifecycle phase?

| Question mentions... | Phase | Domain |
|---|---|---|
| Schema design, data types, SQL syntax, NoSQL choice | DESIGN | 1 |
| Installation, prerequisites, importing, connectivity, testing | BUILD | 2 |
| Monitoring, optimization, patching, change management, documentation | OPERATE | 3 |
| Encryption, access control, attacks, compliance, masking | PROTECT | 4 |
| Backup, replication, failover, RPO/RTO, disaster recovery | SURVIVE | 5 |

### Filter 2: What property is at risk?

| If the question is about... | Property | Look for answers involving... |
|---|---|---|
| Someone seeing data they shouldn't | Confidentiality | Encryption, masking, access control, views |
| Data being wrong or corrupted | Integrity | ACID, constraints, consistency checks, parameterized queries |
| System being down or slow | Availability | Monitoring, HA, backups, load balancing, patching |

### Filter 3: Is this a boundary question?

If two answer choices look very similar, the exam is testing a boundary. Find the distinguishing word:

| If you see... | The distinguishing word is... |
|---|---|
| TRUNCATE vs DELETE | DDL vs DML (or: WHERE clause vs no WHERE) |
| Differential vs Incremental | "last full" vs "last backup of any type" |
| View vs Materialized View | "always current" vs "stale until refreshed" |
| RPO vs RTO | "data loss" vs "downtime" |
| Trigger vs Stored Procedure | "automatic" vs "called by name" |
| Static vs Dynamic masking | "on a copy" vs "at query time" |
| Logical vs Physical schema | "platform-independent" vs "platform-specific" |

### Filter 4: Is this a "first step" question?

If the question asks what to do "first" or "initially," the answer is always in the **assessment** category, never the **action** category. Review, check, compare to baseline, gather requirements — never jump to implementing a fix.

---

## 8. The Ripple Effect — Tracing Consequences

Use these "if...then..." chains to understand how one event cascades through the system. The exam tests whether you can predict downstream effects.

| If this happens... | Then these things are affected... |
|---|---|
| **Backup fails silently** | → RPO is no longer met → DR plan is unreliable → Compliance violation (audit logs) → Data loss risk on failure |
| **Replication lag increases** | → Replica data is stale → Read queries return outdated results → Failover would lose recent transactions → RPO violated |
| **Service account password expires** | → Application can't connect → Users experience outage → Availability impact → Possible cascading failures in dependent systems |
| **Index becomes heavily fragmented** | → Query performance degrades → CPU spikes → Throughput drops → Users experience slowness → More connections pile up |
| **ORM generates N+1 queries** | → DB receives 100x expected query volume → CPU/memory spike → Lock contention increases → Deadlocks possible → Other apps affected |
| **Patch is applied without testing** | → Breaking change in production → Application errors → Potential downtime → Change management violation → Rollback needed |
| **Data discovery is skipped** | → Sensitive data locations unknown → Masking is incomplete → Data refresh exposes PII in dev → Compliance violation |
| **Encryption key is lost** | → Encrypted data is permanently unreadable → If intentional: cryptographic erasure → If accidental: data loss equivalent to destruction |

### Visual: The N+1 Query Explosion

Why the ORM anti-pattern is a system-wide problem, not just a performance issue:

```
CORRECT: 1 query with JOIN                BAD: N+1 pattern (ORM default)

  App ──── 1 query ────► DB                 App ──── 1 query ────► DB
           SELECT orders                             SELECT orders
           JOIN customers                             (returns 100 orders)
           (returns 100 rows)                                │
                                                             ├── query 2 ──► get customer for order 1
  Total: 1 query                                             ├── query 3 ──► get customer for order 2
  DB load: minimal                                           ├── query 4 ──► get customer for order 3
                                                             │   ...
                                                             └── query 101 ► get customer for order 100

                                              Total: 101 queries (1 + N)
                                              DB load: 100x expected

                                              Cascade:
                                              101 queries → connection pool exhausted
                                                → CPU spike (no index can fix this)
                                                → lock contention between queries
                                                → OTHER apps sharing this DB suffer
                                                → deadlocks possible
                                                → monitoring alerts fire

  FIX: Review ORM-generated SQL (always the first step)
       Then use eager loading / explicit JOINs
```

### Visual: Silent Failure Cascade

How a single undetected failure compounds across domains:

```
  Day 1              Day 5              Day 14             Day 30
  ─────────────────────────────────────────────────────────────────►

  Backup job          Nobody             DBA notices        Disk failure.
  starts              notices.           backup files       Need to restore.
  failing             No alert           are 0 bytes.
  silently.           because no         Investigates.      Last valid backup
  ┌─────┐             monitoring         ┌─────┐           is 30 DAYS old.
  │ ✗   │             baseline was       │ !   │           ┌─────┐
  │Fail │             established.       │Found│           │ !!!  │
  └─────┘             ┌─────┐            └─────┘           │LOSS │
                      │ ?   │                              └─────┘
                      │Quiet│            RPO was 24h.
                      └─────┘            Actual loss:       30 days of
                                         14 days.           data GONE.
                      No baseline (P9)
                      → no alert         No restore         Principle 2:
                      → silent failure   testing (P5)       every layer's
                                         → false            failure cascades
                                         confidence         upward
```

---

## 9. Mental Model Summary — The Complete Map

```
                    ┌─────────────────────────────────┐
                    │         REQUIREMENTS             │
                    │   Users · Storage · Objectives   │
                    └──────────────┬──────────────────┘
                                   │
                    ┌──────────────▼──────────────────┐
                    │     DESIGN (Domain 1)            │
                    │  Schema · SQL · NoSQL · Planning │
                    └──────────────┬──────────────────┘
                                   │
                    ┌──────────────▼──────────────────┐
                    │      BUILD (Domain 2)            │
                    │  Install · Connect · Test · Go   │
                    └──────────────┬──────────────────┘
                                   │
              ┌────────────────────▼────────────────────┐
              │                                         │
   ┌──────────▼──────────┐              ┌───────────────▼──────────┐
   │  OPERATE (Domain 3)  │              │   PROTECT (Domain 4)     │
   │  Monitor · Optimize  │◄────────────►│   Encrypt · Control      │
   │  Patch · Document    │   mutually   │   Classify · Defend      │
   │  Change management   │   dependent  │   Comply · Audit         │
   └──────────┬──────────┘              └───────────────┬──────────┘
              │                                         │
              └────────────────────┬────────────────────┘
                                   │
                    ┌──────────────▼──────────────────┐
                    │     SURVIVE (Domain 5)           │
                    │  Backup · Replicate · Failover   │
                    │  RPO/RTO · Test · Failback       │
                    └──────────────┬──────────────────┘
                                   │
                          lessons learned
                                   │
                    ┌──────────────▼──────────────────┐
                    │      Back to DESIGN              │
                    │  (the system is a loop)          │
                    └─────────────────────────────────┘
```

The system is a **feedback loop**, not a waterfall. DR events produce lessons learned that feed back into design. Monitoring reveals optimization needs. Security audits reveal design flaws. The exam tests whether you understand that these domains are not isolated — they are one system viewed from five angles.
