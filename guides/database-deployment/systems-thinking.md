# Domain 2 — Database Deployment: The Assembly Line

Domain 2 is not two separate objectives about planning and implementation. It is one **assembly line** — a strict sequential pipeline where each phase produces the input for the next, and no phase can begin until its predecessor's output is verified. Deployment is the bridge between DESIGN (Domain 1) and OPERATE (Domain 3). Everything Domain 1 planned gets built here. Everything Domain 3 will monitor and maintain is born here.

Skip a phase, and you build on an unverified foundation. Reorder the phases, and you install software on hardware you don't have. The exam tests whether you understand the sequence, the prerequisites, and the gates between phases.

---

## 1. The Assembly Line

The entire domain is one pipeline. Objective 2.1 covers the left side (planning). Objective 2.2 covers the right side (execution). The exam tests where on this pipeline a question lives.

```
  2.1 PLANNING (blueprint)                     2.2 IMPLEMENTATION (construction)
  ─────────────────────────                     ────────────────────────────────

  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
  │REQUIRE-  │  │ARCHITEC- │  │DESIGN    │  │ACQUIRE   │  │DEPLOY    │  │CONNECT   │
  │MENTS     │─►│TURE      │─►│DOCS      │─►│ASSETS    │─►│PHASES    │─►│          │
  │          │  │          │  │          │  │          │  │          │  │          │
  │Users     │  │Cloud vs  │  │Data dict │  │Hardware  │  │Install   │  │DNS       │
  │Storage   │  │on-prem   │  │ERD       │  │Licenses  │  │Provision │  │Firewall  │
  │Objectives│  │IaaS/PaaS/│  │Cardin-   │  │Network   │  │Import    │  │Ports     │
  │(OLTP/    │  │SaaS      │  │ality     │  │IPs/DNS   │  │Upgrade   │  │Static IP │
  │ OLAP)    │  │Gap       │  │Sys req   │  │Service   │  │Modify    │  │Client/   │
  │          │  │analysis  │  │docs      │  │accounts  │  │          │  │server    │
  └──────────┘  └──────────┘  └──────────┘  └──────────┘  └──────────┘  └──────────┘
                                                                              │
       ┌──────────────────────────────────────────────────────────────────────┘
       │
       ▼
  ┌──────────┐  ┌──────────┐  ┌─────────────────────────────────────────────────┐
  │TEST      │  │VALIDATE  │  │                                                 │
  │          │─►│          │─►│  GO LIVE → hand off to Domain 3 (OPERATE)       │
  │9 test    │  │6 valid-  │  │                                                 │
  │types     │  │ation     │  │  Monitoring begins. Maintenance begins.         │
  │(see §5)  │  │checks    │  │  Change management governs all future changes. │
  │          │  │(see §6)  │  │                                                 │
  └──────────┘  └──────────┘  └─────────────────────────────────────────────────┘

  ◄──── each arrow is a GATE ────►
  Nothing passes to the next phase until the current phase's output is verified.
```

**The key insight:** Every exam question about Domain 2 sits on this pipeline. "What should the DBA do before installation?" = you're at the ACQUIRE gate. "After importing data, NULLs appear in columns" = you're at the VALIDATE gate. Locate the question on the pipeline, and the answer is either the gate you're at or the phase immediately before it.

---

## 2. Governing Principles of Deployment

### Principle 1: Every Phase Has Prerequisites — No Phase Is Optional

The deployment pipeline is a strict dependency chain. Each phase requires verified output from the previous phase:

```
  PHASE                REQUIRES (before you can start)
  ──────────────       ──────────────────────────────────────────────

  Requirements    ──►  Business stakeholders, use case definitions

  Architecture    ──►  Completed requirements (users, storage, objectives)

  Design docs     ──►  Architecture decisions made (cloud model, platform)

  Acquire assets  ──►  System requirements documentation
                       (so you know WHAT to acquire)

  Installation    ──►  Hardware in place
                       OS installed and patched
                       Prerequisites verified (runtime libraries, ports)
                       Service accounts provisioned

  Provisioning    ──►  DBMS installed and configured
                       Storage allocated

  Importing       ──►  Schema created (tables, constraints, relationships)
                       Source data available and mapped

  Testing         ──►  Data imported
                       Code deployed (stored procs, triggers, functions)

  Validation      ──►  Tests completed
                       Results documented

  Go Live         ──►  All validations passed
                       Monitoring configured (→ Domain 3)
                       Backup schedule in place (→ Domain 5)
                       Security controls applied (→ Domain 4)
```

**Exam implication:** "What should the DBA do FIRST?" questions in Domain 2 are asking you to identify the prerequisite that hasn't been met. The answer is always the phase that should have come before the current action.

### Principle 2: Planning Is Design-Time — Implementation Is Build-Time

Objectives 2.1 and 2.2 are not separate topics. They are the two halves of one pipeline. 2.1 produces decisions and documents. 2.2 consumes them.

```
  2.1 PRODUCES                              2.2 CONSUMES
  ────────────                              ────────────

  Requirements ────────────────────────────► Determines hardware sizing,
                                             licensing, cloud model

  Logical schema (ERD, cardinality) ───────► Built into physical schema
                                             during provisioning

  Data dictionary ─────────────────────────► Referenced during every
                                             deployment phase and after

  System requirements doc ─────────────────► Procurement checklist for
                                             asset acquisition

  Architecture decision (IaaS/PaaS/SaaS) ──► Determines what you install
                                             vs what the provider manages

  Gap analysis ────────────────────────────► Drives what to acquire
                                             (hardware, licenses, skills)
```

**The rule:** If a question asks "what should the DBA review before deploying?" — the answer is a 2.1 artifact (ERD, data dictionary, system requirements). If it asks "what should the DBA do during deployment?" — the answer is a 2.2 action (install, provision, import, test).

### Principle 3: Testing and Validation Are Different Activities

The exam treats these as distinct phases with different purposes. Confusing them is a common trap.

```
  TESTING (verification)                    VALIDATION (confirmation)
  ┌──────────────────────────────┐          ┌──────────────────────────────┐
  │                              │          │                              │
  │ "Does the system WORK?"     │          │ "Does it work CORRECTLY?"   │
  │                              │          │                              │
  │ Focus: functionality,        │          │ Focus: accuracy, integrity,  │
  │ performance, resilience      │          │ completeness, scalability    │
  │                              │          │                              │
  │ • Does the code execute?    │          │ • Are data values correct?  │
  │ • Does it handle bad input? │          │ • Do mappings match source? │
  │ • Does it survive load?     │          │ • Are indexes optimized?    │
  │ • Do alerts fire?           │          │ • Is referential integrity  │
  │ • Does schema match design? │          │   enforced (no orphans)?    │
  │                              │          │ • Does it scale?            │
  │ Catches: bugs, syntax errors,│          │                              │
  │ performance problems,        │          │ Catches: data quality issues,│
  │ missing notifications        │          │ mapping errors, missing      │
  │                              │          │ constraints, capacity limits │
  └──────────────────────────────┘          └──────────────────────────────┘
                   │                                         │
                   └────────────── BOTH must pass ───────────┘
                                       │
                                       ▼
                                   GO LIVE
```

### Principle 4: The Import Is the Most Dangerous Phase

Importing data is where the most things can silently go wrong. Every other phase either fails loudly (installation errors, syntax errors) or is structural (provisioning). Import failures can be invisible — wrong data types, truncated values, encoding corruption, missing rows — and they propagate downstream into every query, report, and decision made from the data.

```
  IMPORT RISKS AND THEIR SYMPTOMS:

  ┌────────────────────────────┐     ┌────────────────────────────────┐
  │ RISK                       │     │ HOW IT MANIFESTS               │
  │                            │     │                                │
  │ Encoding mismatch          │────►│ Special characters corrupted   │
  │ (UTF-8 vs Latin-1)        │     │ Names with accents broken      │
  │                            │     │                                │
  │ Data type mismatch         │────►│ Truncated values, silent       │
  │ (VARCHAR too short)        │     │ data loss                      │
  │                            │     │                                │
  │ NULL handling              │────►│ Unexpected NULLs in columns    │
  │ (source uses "" not NULL)  │     │ that should have values        │
  │                            │     │                                │
  │ Row count discrepancy      │────►│ Missing records; reports       │
  │ (partial load failure)     │     │ show wrong totals              │
  │                            │     │                                │
  │ FK violations              │────►│ Orphan records; JOINs          │
  │ (child without parent)     │     │ return fewer rows than expected│
  │                            │     │                                │
  │ Index overhead             │────►│ Bulk load is extremely slow    │
  │ (indexes active during     │     │ because every INSERT updates   │
  │  bulk insert)              │     │ every index                    │
  └────────────────────────────┘     └────────────────────────────────┘

  BEST PRACTICE: Disable indexes → Bulk load → Validate → Rebuild indexes
```

### Principle 5: Connectivity Is Architecture, Not Configuration

The networking decisions in 2.2 are not just "know the port numbers." They define the physical security boundaries and failure modes of the entire system:

```
  EVERY CONNECTIVITY DECISION HAS A SECURITY CONSEQUENCE:

  Decision                        Security Consequence
  ────────────────────────        ─────────────────────────────────

  Static vs Dynamic IP       ──►  Static = predictable firewall rules
                                  Dynamic = rules break when IP changes
                                  Production: ALWAYS static

  DNS TTL setting            ──►  Low TTL = faster failover (DNS resolves
                                  to new IP quickly)
                                  High TTL = slower failover but less DNS
                                  query overhead

  Which ports are open       ──►  Only DB ports (1433/3306/5432/27017)
                                  between app tier and DB tier
                                  ALL others closed

  DB placement               ──►  NEVER in DMZ (internet-facing)
                                  ALWAYS on internal network segment
                                  Behind firewall, accessible only from
                                  app tier

  Service account for        ──►  Must exist BEFORE installation begins
  DBMS and app connections        Least privilege, secrets manager,
                                  never hardcoded
```

### Principle 6: Deployment Is Reversible Until Go-Live

Every deployment phase has a rollback path — except after go-live, when Domain 3's change management takes over. The exam tests whether you plan for failure:

| Phase | Rollback mechanism |
|---|---|
| Installation | Uninstall; restore from OS snapshot |
| Provisioning | Drop databases; deallocate storage |
| Upgrading | Rollback plan; database backup taken before upgrade |
| Modifying | Revert configuration; restore from pre-change backup |
| Importing | Truncate and re-import from source; restore from pre-import backup |

**The rule:** "What should the DBA prepare before upgrading the DBMS?" → A rollback plan and a full backup. Always.

---

## 3. The Prerequisites Cascade

Each deployment phase produces an artifact that becomes the input for the next phase. This visual shows the entire chain and what breaks if any link is missing.

```
  ┌─────────────────────────────────────────────────────────────────────────┐
  │                   THE PREREQUISITES CASCADE                             │
  │                                                                         │
  │  REQUIREMENTS ──────────────────────────────────────────────────────┐   │
  │  Output: user count, storage needs, OLTP/OLAP, use cases           │   │
  │                                                                     │   │
  │  If skipped: everything that follows is sized wrong,               │   │
  │  platform choice is a guess, schema doesn't match workload         │   │
  │                                                                     ▼   │
  │  ARCHITECTURE ──────────────────────────────────────────────────┐       │
  │  Output: cloud/on-prem decision, IaaS/PaaS/SaaS, gap analysis │       │
  │                                                                 │       │
  │  If skipped: wrong platform, missing hardware, budget overrun  │       │
  │                                                                 ▼       │
  │  DESIGN DOCS ──────────────────────────────────────────────┐           │
  │  Output: ERD, data dictionary, cardinality, sys req docs   │           │
  │                                                             │           │
  │  If skipped: schema built on assumptions, no shared         │           │
  │  reference, deployment team doesn't know what to build      ▼           │
  │  ACQUIRE ASSETS ──────────────────────────────────────┐                 │
  │  Output: hardware, licenses, IPs, DNS, service accts  │                 │
  │                                                        │                 │
  │  If skipped: installation fails immediately            ▼                 │
  │  INSTALL & CONFIGURE ─────────────────────────────┐                     │
  │  Output: running DBMS instance                     │                     │
  │                                                     │                     │
  │  If prerequisites not met: installation fails      ▼                     │
  │  PROVISION ────────────────────────────────────┐                         │
  │  Output: databases, schemas, users, storage    │                         │
  │                                                 │                         │
  │  If schema wrong: import will fail or produce  ▼                         │
  │  IMPORT ───────────────────────────────────┐                             │
  │  Output: populated tables                   │                             │
  │                                              │                             │
  │  If not validated: silent data quality      ▼                             │
  │  problems propagate into production    TEST & VALIDATE                   │
  │                                        Output: verified, correct system  │
  │                                                                           │
  └─────────────────────────────────────────────────────────────────────────┘
```

---

## 4. The Connectivity Architecture

The networking section of 2.2 defines where the database physically sits relative to users, applications, and the internet. This is not configuration trivia — it's the security boundary of the entire system.

```
  INTERNET
      │
      │  ┌─────────────────────────────────────────────────────────────┐
      │  │                                                             │
      │  │  DMZ (Perimeter Network)                                    │
      │  │  ┌───────────────────────────────────────────────────────┐  │
      │  │  │                                                       │  │
      │  │  │  Web servers · Load balancers · WAF                   │  │
      │  │  │                                                       │  │
      │  │  │  DATABASE: ██ NEVER HERE ██                           │  │
      │  │  │                                                       │  │
      │  │  └───────────────────────┬───────────────────────────────┘  │
      │  │                          │                                   │
      │  │                    FIREWALL RULES:                          │
      │  │                    Allow ONLY app tier → DB tier            │
      │  │                    On DB ports ONLY                         │
      │  │                          │                                   │
      │  │  ┌───────────────────────▼───────────────────────────────┐  │
      │  │  │                                                       │  │
      │  │  │  APPLICATION TIER                                     │  │
      │  │  │  App servers · Connection pools · Service accounts    │  │
      │  │  │                                                       │  │
      │  │  └───────────────────────┬───────────────────────────────┘  │
      │  │                          │                                   │
      │  │                    DB PORTS ONLY:                            │
      │  │                    ┌────────────────────────────────┐       │
      │  │                    │ SQL Server    1433/TCP         │       │
      │  │                    │ MySQL         3306/TCP         │       │
      │  │                    │ PostgreSQL    5432/TCP         │       │
      │  │                    │ MongoDB       27017/TCP        │       │
      │  │                    └────────────────────────────────┘       │
      │  │                          │                                   │
      │  │  ┌───────────────────────▼───────────────────────────────┐  │
      │  │  │                                                       │  │
      │  │  │  DATABASE TIER (Internal Network / VLAN / VPC)        │  │
      │  │  │                                                       │  │
      │  │  │  ┌──────────────────────────────────────────────┐    │  │
      │  │  │  │  DB SERVER                                    │    │  │
      │  │  │  │                                               │    │  │
      │  │  │  │  Static IP (predictable firewall rules)       │    │  │
      │  │  │  │  TLS/SSL on all connections                   │    │  │
      │  │  │  │  Service account with least privilege         │    │  │
      │  │  │  │  No internet exposure                         │    │  │
      │  │  │  │  DNS A/CNAME record points here               │    │  │
      │  │  │  │  TTL: low for faster failover                 │    │  │
      │  │  │  └──────────────────────────────────────────────┘    │  │
      │  │  │                                                       │  │
      │  │  └───────────────────────────────────────────────────────┘  │
      │  │                                                             │
      │  └─────────────────────────────────────────────────────────────┘
```

### The DNS–Failover Connection

DNS isn't just name resolution. Its TTL setting directly controls failover speed — a concept that bridges Domain 2 (connectivity) and Domain 5 (business continuity):

```
  NORMAL OPERATION:

  App resolves db.company.com ──► DNS returns 10.0.1.50 ──► App connects
                                  (cached for TTL duration)


  FAILOVER (primary dies, standby at 10.0.2.50):

  HIGH TTL (e.g., 24 hours)                 LOW TTL (e.g., 60 seconds)
  ┌──────────────────────────┐              ┌──────────────────────────┐
  │ DBA updates DNS to       │              │ DBA updates DNS to       │
  │ point to 10.0.2.50       │              │ point to 10.0.2.50       │
  │                          │              │                          │
  │ Apps keep using cached   │              │ Apps refresh within      │
  │ 10.0.1.50 for up to     │              │ 60 seconds → connect     │
  │ 24 HOURS                 │              │ to standby quickly       │
  │                          │              │                          │
  │ Result: extended outage  │              │ Result: fast failover    │
  │ despite standby being    │              │ (minutes, not hours)     │
  │ ready                    │              │                          │
  └──────────────────────────┘              └──────────────────────────┘

  Trade-off: low TTL = more DNS queries (slightly more network overhead)
             but dramatically faster disaster recovery
```

---

## 5. The Testing Taxonomy

Domain 2 defines nine distinct test types. They are not a random list — they form a progressive system that validates the deployment from structure to function to resilience to process.

```
  PROGRESSION: STRUCTURAL → FUNCTIONAL → LOAD → RESILIENCE → PROCESS

  STRUCTURAL (does it exist correctly?)
  ┌──────────────────────────────────────────────────────────────────────┐
  │                                                                      │
  │  1. DATABASE QUALITY CHECK                                          │
  │     Columns, tables, fields match the design spec.                  │
  │     "Do the physical objects match the logical schema?"             │
  │                                                                      │
  │  2. SCHEMA MEETS ORIGINAL REQUIREMENTS                             │
  │     Logical → physical mapping is complete and correct.             │
  │     "Did we build what was designed?"                               │
  │                                                                      │
  │  3. SYNTAX ERRORS                                                   │
  │     SQL scripts parse and execute without errors.                   │
  │     "Does the code compile?"                                        │
  │                                                                      │
  └──────────────────────────────────────────────────────────────────────┘

  FUNCTIONAL (does it work?)
  ┌──────────────────────────────────────────────────────────────────────┐
  │                                                                      │
  │  4. CODE EXECUTION                                                  │
  │     Stored procedures, functions, triggers run without errors.      │
  │     "Do the programming objects produce correct results?"           │
  │                                                                      │
  │  5. NOTIFICATION TRIGGERS AND ALERTS                                │
  │     Email, SNMP, webhook notifications fire on thresholds.          │
  │     "Does the alerting system work before we need it?"             │
  │                                                                      │
  └──────────────────────────────────────────────────────────────────────┘

  LOAD (does it survive pressure?)
  ┌──────────────────────────────────────────────────────────────────────┐
  │                                                                      │
  │  6. STRESS TESTING                                                  │
  │     Two forms:                                                      │
  │     • Stored procedure stress test: sustained high call volume      │
  │       against specific procedures                                   │
  │     • Application stress test: simulated user load against the      │
  │       full stack (app + DB together)                                │
  │     "Does it hold up under production-like load?"                  │
  │                                                                      │
  └──────────────────────────────────────────────────────────────────────┘

  RESILIENCE (does it handle failure gracefully?)
  ┌──────────────────────────────────────────────────────────────────────┐
  │                                                                      │
  │  7. NEGATIVE TESTING                                                │
  │     Deliberately send BAD data: constraint violations, wrong types, │
  │     null where not allowed, overlength strings, SQL injection       │
  │     patterns.                                                       │
  │     "Does the system REJECT what it should reject?"                │
  │                                                                      │
  │     ⚠ EXAM TRAP: Negative testing is NOT performance testing.      │
  │     It tests error handling, not speed.                             │
  │                                                                      │
  └──────────────────────────────────────────────────────────────────────┘

  PROCESS (does the development workflow hold up?)
  ┌──────────────────────────────────────────────────────────────────────┐
  │                                                                      │
  │  8. VERSION CONTROL TESTING                                         │
  │     Schema scripts are in source control. Migrations apply cleanly  │
  │     from a clean state. "Can we rebuild this from scratch?"        │
  │                                                                      │
  │  9. REGRESSION TESTING                                              │
  │     After any change, existing functionality still works.           │
  │     "Did the new feature break anything old?"                      │
  │                                                                      │
  └──────────────────────────────────────────────────────────────────────┘
```

### The Test Selection Decision Rule

```
  Question describes...                                Test type

  "Tables and columns don't match the design"    ──►  Database quality check
  "Physical schema doesn't reflect the ERD"      ──►  Schema meets requirements
  "SQL script fails with parse error"            ──►  Syntax errors
  "Stored procedure returns wrong results"       ──►  Code execution
  "Alert didn't fire when disk hit threshold"    ──►  Notification triggers/alerts
  "System slows under 500 concurrent users"      ──►  Stress testing
  "Invalid input accepted without error"         ──►  Negative testing
  "Migration script fails on fresh install"      ──►  Version control testing
  "New feature broke existing report"            ──►  Regression testing
```

---

## 6. The Validation Gate — Bridge to Domain 3

Validation is the final checkpoint before go-live. It's where you prove the system is not just functional (testing) but **correct and complete** (validation). Six specific checks, each with a different purpose:

```
  ┌──────────────────────────────────────────────────────────────────────────┐
  │                      THE SIX VALIDATION CHECKS                          │
  │                                                                          │
  │  ┌──────────────────┐                                                   │
  │  │ 1. INDEX ANALYSIS │  Are indexes present for frequent query patterns?│
  │  │                   │  Any duplicate or unused indexes?                │
  │  │  PURPOSE:         │  Missing index → table scans in production →    │
  │  │  Performance      │  CPU/IO problems from day one                   │
  │  │  readiness        │                                                  │
  │  └──────────────────┘                                                   │
  │                                                                          │
  │  ┌──────────────────┐                                                   │
  │  │ 2. DATA MAPPING  │  Does each source field map to the correct       │
  │  │                   │  target column? Types compatible?                │
  │  │  PURPOSE:         │  Wrong mapping → customer_name in email column  │
  │  │  Structural       │  → every downstream query returns wrong data    │
  │  │  accuracy         │                                                  │
  │  └──────────────────┘                                                   │
  │                                                                          │
  │  ┌──────────────────┐                                                   │
  │  │ 3. DATA VALUES   │  Spot-check actual values: NULLs where expected?│
  │  │                   │  Truncation? Encoding issues? Expected ranges?  │
  │  │  PURPOSE:         │  Catches silent import failures — the CSV had   │
  │  │  Content          │  "N/A" where the column expects NULL            │
  │  │  accuracy         │                                                  │
  │  └──────────────────┘                                                   │
  │                                                                          │
  │  ┌──────────────────┐                                                   │
  │  │ 4. QUERIES       │  Key queries return correct results within       │
  │  │                   │  acceptable response time.                       │
  │  │  PURPOSE:         │  End-to-end proof that schema + data + indexes  │
  │  │  Functional       │  produce the right answers fast enough          │
  │  │  proof            │                                                  │
  │  └──────────────────┘                                                   │
  │                                                                          │
  │  ┌──────────────────┐                                                   │
  │  │ 5. REFERENTIAL   │  Foreign keys enforced. No orphan records.       │
  │  │    INTEGRITY     │  Row counts match expected relationships.        │
  │  │                   │                                                  │
  │  │  PURPOSE:         │  Orphan orders without customers → JOINs        │
  │  │  Relational       │  silently drop rows → reports are wrong →       │
  │  │  correctness      │  nobody notices until audit                     │
  │  └──────────────────┘                                                   │
  │                                                                          │
  │  ┌──────────────────┐                                                   │
  │  │ 6. SCALABILITY   │  Increase data volume and user count beyond      │
  │  │    VALIDATION    │  current projections. Does it still perform?     │
  │  │                   │                                                  │
  │  │  PURPOSE:         │  Proves the system won't collapse on month 3    │
  │  │  Future-          │  when data doubles or users triple              │
  │  │  proofing         │                                                  │
  │  └──────────────────┘                                                   │
  │                                                                          │
  │  ALL SIX PASS?                                                          │
  │      │                                                                   │
  │      YES ──► GO LIVE                                                    │
  │              Configure monitoring (D3) · Enable backups (D5)            │
  │              Apply security controls (D4)                               │
  │                                                                          │
  │      NO ───► TRACE BACK to the pipeline phase that produced the         │
  │              bad output. Fix at the source, not at the symptom.         │
  │                                                                          │
  └──────────────────────────────────────────────────────────────────────────┘
```

### Visual: Validation Failure Trace-Back

When a validation check fails, the root cause is always in an earlier pipeline phase:

```
  VALIDATION FAILURE              ROOT CAUSE PHASE         FIX

  Index analysis: missing    ──►  Provisioning             Add indexes to schema scripts
  indexes for key queries         (schema creation)         Re-run provisioning

  Data mapping: field in     ──►  Importing                Fix mapping definition
  wrong column                    (ETL/load script)         Re-import

  Data values: unexpected    ──►  Importing                Fix source data or
  NULLs                           (source data quality)     transformation logic

  Queries: wrong results     ──►  Provisioning             Fix schema (constraints,
                                  (schema design)           relationships, views)

  Referential integrity:     ──►  Importing                Load parent records before
  orphan records                  (load order)              child records; enforce FKs

  Scalability: performance   ──►  Architecture             Storage tier insufficient;
  degrades at 2x volume           (capacity planning)       need SAN/SSD or scale-out
```

---

## 7. The Deployment Phases — Internal Mechanics

### Visual: The Five Deployment Sub-Phases

Each sub-phase of deployment (installation, provisioning, upgrading, modifying, importing) has a distinct purpose and a distinct risk:

```
  ┌─────────────────────────────────────────────────────────────────────┐
  │  INSTALLATION & CONFIGURATION                                      │
  │                                                                     │
  │  Input:  Hardware + OS + prerequisites verified                    │
  │  Action: Install DBMS binaries; set instance parameters            │
  │          (memory allocation, tempdb config, listener ports)        │
  │  Output: Running DBMS instance (empty)                             │
  │                                                                     │
  │  Key check: DATABASE PREREQUISITES                                 │
  │  ┌─────────────────────────────────────────────────────────┐       │
  │  │ OS version compatible? ✓                                │       │
  │  │ OS patches current? ✓                                   │       │
  │  │ Runtime libraries installed (.NET, Java, etc.)? ✓       │       │
  │  │ Sufficient disk/RAM/CPU? ✓                              │       │
  │  │ Service account created? ✓                              │       │
  │  │ Network ports available? ✓                              │       │
  │  │ Firewall rules configured? ✓                            │       │
  │  └─────────────────────────────────────────────────────────┘       │
  │  If ANY prerequisite fails → installation fails or behaves        │
  │  unpredictably                                                     │
  ├─────────────────────────────────────────────────────────────────────┤
  │  PROVISIONING                                                      │
  │                                                                     │
  │  Input:  Running DBMS instance                                     │
  │  Action: Create databases, schemas, tables, constraints, users     │
  │          Allocate storage (filegroups, tablespaces)                │
  │  Output: Empty schema ready for data                               │
  │                                                                     │
  │  Best practice: Script everything. Use IaC for repeatability.      │
  │  Automate so dev/staging/production are built identically.         │
  ├─────────────────────────────────────────────────────────────────────┤
  │  UPGRADING                                                         │
  │                                                                     │
  │  Input:  Running system on older DBMS version                      │
  │  Action: Upgrade to newer DBMS version                             │
  │  Output: System running on new version                             │
  │                                                                     │
  │  REQUIRES: Compatibility testing, deprecated-feature audit,        │
  │  rollback plan, FULL BACKUP before upgrade                         │
  │  Governed by: change management (D3)                               │
  ├─────────────────────────────────────────────────────────────────────┤
  │  MODIFYING                                                         │
  │                                                                     │
  │  Input:  Running system needing configuration change               │
  │  Action: Add filegroups, change collation, resize, alter config   │
  │  Output: Modified system                                           │
  │                                                                     │
  │  REQUIRES: Change management approval (D3)                         │
  │  ALWAYS: Test in non-production first                              │
  ├─────────────────────────────────────────────────────────────────────┤
  │  IMPORTING                                                         │
  │                                                                     │
  │  Input:  Empty schema + source data (CSV, flat files, other DB)   │
  │  Action: Load data; transform as needed; validate                  │
  │  Output: Populated database                                        │
  │                                                                     │
  │  Performance tip: Disable indexes → Bulk load → Validate →        │
  │  Rebuild indexes (avoids index overhead during mass INSERT)        │
  │                                                                     │
  │  VALIDATE IMMEDIATELY:                                             │
  │  • Row counts match source                                        │
  │  • Data types didn't truncate values                               │
  │  • Encoding preserved (UTF-8 characters intact)                   │
  │  • NULLs are intentional, not import artifacts                    │
  │  • Referential integrity holds (no orphan records)                │
  └─────────────────────────────────────────────────────────────────────┘
```

---

## 8. Where Domain 2 Touches Every Other Domain

Deployment is the bridge. It consumes Domain 1's output and produces Domain 3/4/5's input:

```
  DOMAIN 1 (DESIGN)                DOMAIN 2 (BUILD)               DOMAINS 3, 4, 5
  ─────────────────                ────────────────               ─────────────────

  Logical schema (ERD,      ──►    Built into physical    ──►    Maintained via change
  cardinality, data dict)          schema during                  management (D3)
                                   provisioning                   Updated data dict (D3)

  SQL sublanguages           ──►   Code execution testing  ──►   Query optimization (D3)
  (DDL, DML, TCL)                  during deployment              Execution plan analysis

  Programming objects        ──►   Stress-tested during    ──►   Optimized during
  (triggers, procs, views)         deployment                     maintenance (D3)

  Storage architecture       ──►   Hardware acquired and   ──►   Monitored for capacity
  (SSD, SAN, IOPS)                 provisioned                    and growth (D3)

  Cloud model (IaaS/         ──►   Determines what DBA     ──►   Affects patching
  PaaS/SaaS)                       installs vs what               responsibility (D3)
                                   provider manages               and security scope (D4)

  Requirements               ──►   Service accounts        ──►   Least privilege
  (users, access)                  provisioned before             enforced (D4)
                                   deployment                     Rotated periodically (D4)

  N/A                        ──►   Connectivity (DNS,      ──►   Firewall rules
                                   ports, firewall)               reviewed (D4)
                                   configured                     DNS TTL affects
                                                                  failover speed (D5)
```

---

## 9. Domain 2 Decision Engine

### Filter 1: Where on the assembly line?

| Question is about... | Pipeline position | Sub-objective |
|---|---|---|
| Users, storage needs, OLTP/OLAP, gap analysis, cloud model | Planning | 2.1 |
| Schema levels (logical, physical, view), ERD, data dictionary | Design docs | 2.1 |
| Hardware, licenses, network allocations, service accounts | Asset acquisition | 2.2 |
| Installation, provisioning, upgrading, modifying, importing | Deployment phases | 2.2 |
| DNS, IP addressing, firewall, ports, client/server architecture | Connectivity | 2.2 |
| Quality check, stress test, negative test, regression, code execution | Testing | 2.2 |
| Index analysis, data mapping, data values, referential integrity | Validation | 2.2 |

### Filter 2: Is it a "what first?" question?

| Scenario | First step | Why |
|---|---|---|
| "DBA is about to install DBMS" | Verify prerequisites (OS, patches, runtime, service account) | Can't install on unprepared infrastructure |
| "DBA is about to import data" | Confirm schema exists and source data is mapped | Can't load into nonexistent tables |
| "DBA is about to upgrade DBMS" | Full backup + rollback plan | Must be able to reverse if upgrade fails |
| "DBA wants to modify production config" | Change management approval | No production changes without CAB sign-off |
| "DBA is planning a new database" | Requirements gathering (users, storage, objectives) | Can't design without knowing the workload |
| "After import, columns have unexpected NULLs" | Validate data values and data mapping | Identify whether NULLs are from source or import error |
| "After deployment, queries are slow" | Index analysis | Most common root cause of post-deployment performance issues |

### Filter 3: Is it a boundary question?

| Pair | Distinguishing property |
|---|---|
| Testing vs Validation | Testing = "does it work?" Validation = "is it correct?" |
| Database quality check vs Schema meets requirements | Quality check = objects exist. Schema meets requirements = objects match the ERD/design |
| Stress testing vs Negative testing | Stress = high load performance. Negative = bad input rejection |
| Regression testing vs Version control testing | Regression = old features still work after changes. Version control = scripts apply cleanly from scratch |
| Stored procedure stress test vs Application stress test | Procedure = sustained calls to specific DB routines. Application = simulated users against full stack |
| Provisioning vs Modifying | Provisioning = creating new. Modifying = changing existing |
| Upgrading vs Modifying | Upgrading = new DBMS version. Modifying = configuration change on same version |
| Static vs Dynamic IP | Static = production (predictable rules). Dynamic = dev/test only |
| Data mapping vs Data values | Mapping = field-to-column correctness. Values = actual content correctness |
| Index analysis vs Scalability validation | Index analysis = current performance. Scalability = future performance under growth |
| Logical vs Physical schema | Platform-independent design vs platform-specific implementation |

### Filter 4: Is it a test type identification question?

| Scenario description | Test type |
|---|---|
| "Columns in the deployed database don't match the design document" | Database quality check |
| "Physical schema is missing entities from the ERD" | Schema meets original requirements |
| "SQL script fails with error on line 42" | Syntax errors |
| "Stored procedure returns 0 rows instead of expected results" | Code execution |
| "Email alert didn't fire when disk usage hit 90%" | Notification triggers and alerts |
| "Database response time degrades under 500 concurrent users" | Stress testing (application) |
| "Stored procedure timeout under sustained rapid execution" | Stress testing (stored procedure) |
| "Form accepts `' OR 1=1 --` without error" | Negative testing |
| "Schema migration fails when run against an empty instance" | Version control testing |
| "New column addition broke the monthly financial report" | Regression testing |

---

## 10. Domain 2 Ripple Effects

| If this deployment step fails... | Cascade through the system... |
|---|---|
| **Requirements skipped** | → Architecture decision is a guess → Wrong cloud model or hardware → Storage insufficient within months → Performance degrades → Emergency capacity upgrade → Downtime during migration |
| **Gap analysis skipped** | → Missing hardware or licenses discovered during installation → Deployment delayed → Business deadlines missed → Pressure to skip testing → Bugs reach production |
| **Prerequisites not verified before install** | → Installation fails or produces unstable DBMS → Intermittent crashes → Difficult-to-diagnose production issues → Eventual reinstallation required under time pressure |
| **Service accounts not provisioned before deployment** | → Applications can't connect → Emergency use of admin credentials → Least privilege violated → Security audit finding → Must retroactively create proper accounts and rotate compromised credentials |
| **Importing without disabling indexes** | → Bulk load takes 10x longer → Maintenance window exceeded → Partial load → Must restart → Deadline slips |
| **Import not validated (data values)** | → Silent data quality problems → Reports show wrong numbers → Business decisions made on bad data → Discovered weeks later → Credibility loss → Expensive manual data correction |
| **Referential integrity not validated** | → Orphan records exist → JOINs silently drop rows → Aggregations undercount → Financial reports are wrong → Audit discovers discrepancy → Retroactive data repair required |
| **Negative testing skipped** | → Invalid inputs accepted in production → Constraint violations surface as application crashes → SQL injection vulnerability goes undetected → Potential breach |
| **Stress testing skipped** | → System works perfectly with 10 users → Collapses at 200 users on launch day → Emergency scaling under load → Extended outage → Users migrate to competitor |
| **DNS TTL set too high** | → Failover event occurs → Standby is ready within seconds → But apps keep connecting to dead primary for hours → RTO violated despite having infrastructure in place → DR plan fails at the network layer |
| **Static IP not used for production DB** | → DHCP assigns new IP → Firewall rules break → Application connection strings point to old IP → Outage → DBA scrambles to find new IP and update everything → Preventable downtime |
