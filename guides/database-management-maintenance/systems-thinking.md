# Domain 3 — Database Management & Maintenance: The Operations System

Domain 3 is not four separate objectives about monitoring, maintenance, documentation, and data tasks. It is one **continuous operations loop** — a feedback system where signals drive diagnosis, diagnosis drives action, action requires validation, and validation produces the documentation that makes the next cycle faster. Every concept in Domain 3 is either a sensor, a response, a process gate, or a record.

This is the highest-weighted domain (25%). The exam invests here because operations is where theory becomes reality — where every design choice from Domain 1, every deployment decision from Domain 2, every security control from Domain 4, and every continuity plan from Domain 5 is either validated or exposed as insufficient.

---

## 1. The Operations Loop

Domain 3 runs on a single feedback loop. Every monitoring metric, every maintenance process, every document, and every data management task maps to a position on this loop.

```
                    ┌──────────────────────────────────┐
                    │           MONITOR (3.1)           │
                    │  Collect signals: CPU, memory,    │
                    │  disk, throughput, replication,    │
                    │  logs, connections, backups        │
                    └───────────────┬──────────────────┘
                                    │ compare to baseline
                                    ▼
                    ┌──────────────────────────────────┐
                    │           DETECT                  │
                    │  Is this metric outside the       │
                    │  baseline? Fire alert if yes.     │
                    │  No baseline → can't detect.      │
                    └───────────────┬──────────────────┘
                                    │ anomaly confirmed
                                    ▼
                    ┌──────────────────────────────────┐
                    │          DIAGNOSE                  │
                    │  Trace root cause: execution       │
                    │  plans, deadlock graphs, log       │
                    │  correlation, index analysis       │
                    └───────────────┬──────────────────┘
                                    │ root cause identified
                                    ▼
  ┌─────────────────────────────────────────────────────────────────┐
  │                    CHANGE MANAGEMENT (3.2)                      │
  │  Plan → Approve (CAB) → Communicate → Implement → Validate     │
  │                                                                 │
  │  ALL actions pass through this gate:                            │
  │  • Query/index optimization     • Patch application            │
  │  • Schema changes               • Database refresh             │
  │  • Capacity upgrades            • Vulnerability remediation    │
  └─────────────────────────────────┬───────────────────────────────┘
                                    │ change validated
                                    ▼
                    ┌──────────────────────────────────┐
                    │          DOCUMENT (3.3)            │
                    │  Update: data dictionary, ERD,    │
                    │  maintenance docs, SOPs,          │
                    │  compliance records                │
                    └───────────────┬──────────────────┘
                                    │ feeds back into
                                    ▼
                              MONITOR (3.1)
                              (loop restarts)
```

**The key insight:** The four sub-objectives are not four topics — they are four positions on one loop. Monitoring (3.1) feeds maintenance (3.2). Maintenance produces documentation (3.3). Data management tasks (3.4) are the specific actions performed during the "Implement" phase. Change management wraps around everything.

---

## 2. Governing Principles of Database Operations

### Principle 1: A Metric Without a Baseline Is Noise

This is the single most testable concept in Domain 3. The exam will describe a metric value and ask whether it's a problem.

```
  WITHOUT BASELINE                          WITH BASELINE

  "CPU is at 82%"                           "CPU is at 82%"
       │                                         │
       ▼                                         ▼
  Is this bad?                              Baseline is 45%
  ¯\_(ツ)_/¯                                     │
  No way to know.                                ▼
                                            82% vs 45% = anomaly
                                                 │
                                                 ▼
                                            Alert fires.
                                            DBA investigates.
                                            Finds: missing index
                                            on new table causing
                                            full table scans.
```

The chain: **Baseline → Threshold → Alert → Investigation → Root cause → Fix**

Remove the baseline and the entire chain collapses. Every monitoring metric in 3.1 — CPU, memory, disk, throughput, replication lag, connections — is meaningless without this reference point.

**Trending** extends the baseline over time. A metric that's 5% above baseline today and 10% above next month reveals gradual degradation that a single snapshot would miss.

### Principle 2: There Are Two Clocks in Operations

Every process in Domain 3 runs on one of two temporal patterns:

```
  CONTINUOUS (real-time)                    PERIODIC (scheduled)
  ┌──────────────────────────┐              ┌──────────────────────────┐
  │ Always running, always   │              │ Runs on a schedule,      │
  │ collecting                │              │ during maintenance       │
  │                          │              │ windows                  │
  │ • CPU/memory/disk        │              │                          │
  │   monitoring             │              │ • Index rebuild /        │
  │ • Replication lag        │              │   reorganize             │
  │   tracking               │              │ • Patch application      │
  │ • Connection monitoring  │              │ • Integrity checks       │
  │ • Deadlock detection     │              │   (DBCC CHECKDB)         │
  │ • Transaction log growth │              │ • Audit log review       │
  │ • Backup job monitoring  │              │ • Capacity planning      │
  │                          │              │ • Backup scheduling      │
  │ Alert on: threshold      │              │ • Database refresh       │
  │ breach in real time      │              │                          │
  │                          │              │ Governed by: release     │
  │ Failure mode: silent     │              │ schedules, maintenance   │
  │ degradation if not       │              │ windows, compliance      │
  │ configured               │              │ calendars                │
  └──────────────────────────┘              └──────────────────────────┘
```

**Exam implication:** If a question asks "when should the DBA do X?" — determine which clock it belongs to. Monitoring is continuous. Maintenance is periodic. Mixing them up is a common trap.

### Principle 3: Every Maintenance Action Is a Change

No maintenance action — no matter how routine — should bypass change management in production. The exam tests this boundary:

| Feels routine | Still requires change management? | Why |
|---|---|---|
| Rebuilding a fragmented index | Yes | Requires table locks; can impact running queries |
| Applying a security patch | Yes | Can break application compatibility |
| Adding a covering index | Yes | Alters query plans; changes write overhead |
| Updating statistics | Depends on environment | Often automated, but manual runs in production should be documented |
| Database refresh to dev | Yes | Involves sensitive data; must mask before copying down |

The only exception: changes in dev/test environments, where process is intentionally light.

### Principle 4: Logs Are the Database's Nervous System

Domain 3 has three distinct log types. They serve different purposes and the exam tests whether you know which is which:

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                     THE LOG ECOSYSTEM                            │
  │                                                                  │
  │  TRANSACTION LOGS              SYSTEM LOGS         AUDIT LOGS    │
  │  ┌───────────────────┐  ┌──────────────────┐  ┌──────────────┐  │
  │  │ Every data change  │  │ DBMS + OS events │  │ Who did what │  │
  │  │ (INSERT, UPDATE,   │  │                  │  │ and when     │  │
  │  │  DELETE) recorded  │  │ Errors, warnings │  │              │  │
  │  │                   │  │ startup/shutdown  │  │ Schema       │  │
  │  │ PURPOSE:           │  │ auth failures    │  │ changes,     │  │
  │  │ • Point-in-time   │  │                  │  │ data access, │  │
  │  │   recovery         │  │ PURPOSE:         │  │ permission   │  │
  │  │ • Rollback         │  │ • Troubleshoot  │  │ changes      │  │
  │  │ • Replication      │  │ • Detect errors │  │              │  │
  │  │                   │  │ • Security       │  │ PURPOSE:     │  │
  │  │ RISK if ignored:   │  │   (failed logins)│  │ • Compliance │  │
  │  │ Log fills disk →  │  │                  │  │   (PCI, GDPR)│  │
  │  │ writes stop →     │  │ CENTRALIZE via:  │  │ • Forensics  │  │
  │  │ database halts    │  │ SIEM, syslog     │  │ • Detect     │  │
  │  │                   │  │                  │  │   privilege   │  │
  │  │ MANAGE:            │  │                  │  │   escalation │  │
  │  │ Regular log        │  │                  │  │              │  │
  │  │ backups truncate   │  │                  │  │ Reviewed:    │  │
  │  │ the log            │  │                  │  │ Periodically │  │
  │  └───────────────────┘  └──────────────────┘  └──────────────┘  │
  │                                                                  │
  │  All three are DIFFERENT.                                        │
  │  Transaction log = recovery. System log = errors. Audit = who.  │
  └──────────────────────────────────────────────────────────────────┘
```

### Principle 5: Optimization Has a Hierarchy

When diagnosing performance problems, there is a cost-effectiveness order. Start cheap and work down:

```
  LOWEST COST / HIGHEST IMPACT (try first)
  ▲
  │  1. Query optimization
  │     Rewrite the SQL. Eliminate SELECT *, add WHERE clauses,
  │     replace correlated subqueries with JOINs.
  │     Cost: zero. Impact: often dramatic.
  │
  │  2. Index optimization
  │     Add covering indexes for frequent queries.
  │     Rebuild fragmented indexes. Remove unused indexes.
  │     Cost: minimal. Impact: 10x-100x on targeted queries.
  │
  │  3. Execution plan analysis
  │     Identify table scans, key lookups, sort spills.
  │     Guide index and query changes with evidence.
  │     Cost: DBA time only. Impact: eliminates guesswork.
  │
  │  4. Configuration tuning
  │     Buffer pool size, max connections, parallelism settings.
  │     Cost: low. Impact: moderate, system-wide.
  │
  │  5. Load balancing
  │     Route read traffic to replicas. Distribute workload.
  │     Cost: moderate (requires replicas). Impact: scales reads.
  │
  │  6. Hardware upgrade
  │     More RAM, faster storage (SSD/SAN), more CPU.
  │     Cost: HIGH. Impact: raises the ceiling but doesn't fix
  │     bad queries or missing indexes.
  │
  HIGHEST COST / LOWEST RELATIVE IMPACT (last resort)
  ▼
```

**The rule:** The exam favors answers higher on this list. "Add more RAM" is almost never the first answer — "analyze execution plans" or "add an index" comes first.

### Principle 6: Documentation Is the System's Memory

Without documentation, every DBA starts from zero. The exam treats documentation not as paperwork but as **operational infrastructure** — the mechanism that enables consistency, continuity, and compliance.

```
  What happens WITHOUT each document:

  No data dictionary    → Schema changes based on guessing column purposes
                        → Multiple DBAs make conflicting assumptions
                        → Audit fails (can't prove data classification)

  No ERD                → New team members misunderstand relationships
                        → JOINs built on wrong assumptions
                        → Cardinality errors in new tables

  No maintenance docs   → Backup schedule exists only in one DBA's head
                        → That DBA leaves → knowledge lost
                        → Index rebuild frequency is inconsistent

  No SOPs               → Same task done differently by different DBAs
                        → Compliance auditor asks "how do you handle X?"
                        → No answer → finding

  No compliance docs    → Can't prove controls to auditors
                        → PCI DSS / GDPR / HIPAA audit fails
                        → Legal and financial consequences
```

### Principle 7: Redundancy Is a Conscious Trade-Off, Not a Mistake

Data redundancy appears in two forms. The exam tests whether you can distinguish them:

| | Intentional (denormalization) | Unintentional (duplication) |
|---|---|---|
| **Purpose** | Improve read performance | No purpose — design error |
| **Example** | Pre-calculated subtotal stored in order table | Same customer address in 3 tables with no single source of truth |
| **Trade-off** | Accepted: faster reads. Cost: more storage, update anomaly risk | No benefit. Cost: inconsistency, wasted storage, confusion |
| **Exam answer** | Valid design choice for read-heavy workloads | Problem to fix via normalization |

---

## 3. The Signal System — What Every Metric Tells You

Every monitoring metric in 3.1 follows the same architecture: **signal → baseline comparison → threshold → alert → response**. The exam tests whether you can connect the signal to its root cause.

```
  METRIC CATEGORY          WHAT IT SIGNALS                     ROOT CAUSE TO INVESTIGATE
  ─────────────────────────────────────────────────────────────────────────────────────────

  CPU (sustained high)     Compute bottleneck                  Missing indexes → table scans
                                                               Bad query plans → unnecessary sorts
                                                               N+1 ORM queries → 100x query volume
                                                               Insufficient hardware (last resort)

  Memory                   Data not fitting in cache           Buffer pool too small
   (low hit ratio,                                             Too many concurrent queries
    low page life                                              Index bloat consuming cache
    expectancy)                                                Workload exceeds server capacity

  Disk space               Storage exhaustion approaching      Transaction log growing unchecked
                                                               Data growth exceeding capacity plan
                                                               Backup files stored locally
                                                               Temp tables not cleaned up

  Disk I/O latency         Storage performance ceiling         HDD used for active workload (need SSD)
                                                               SAN overloaded by other tenants
                                                               Index fragmentation → random I/O

  Throughput               Processing capacity declining       All of the above + lock contention
   (TPS / QPS declining                                        Deadlocks killing transactions
    under constant load)                                       Connection pool exhaustion

  Replication lag          Replica falling behind primary      Network bandwidth insufficient
                                                               Heavy write activity on primary
                                                               Replica hardware underpowered
                                                               Long-running queries on replica

  Connections              Session pressure                    Connection pool misconfiguration
   (approaching max)                                           Application not closing connections
                                                               N+1 queries consuming pool

  Failed connections       Authentication or capacity issue    Stale credentials in application
                                                               Brute-force attack (D4 crossover)
                                                               Max connections already reached

  Deadlocks                Transaction design conflict         Two+ transactions acquire locks in
                                                               different order on same objects
                                                               Long-running transactions hold locks
                                                               Missing indexes → lock escalation

  Backup size anomaly      Potential data loss or corruption   Dramatically smaller = missing data
                                                               Dramatically larger = unexpected growth
                                                               Zero bytes = job failed silently
```

### Visual: The Performance Degradation Cascade

Performance problems rarely appear in isolation. One bottleneck creates pressure that manifests as symptoms elsewhere. The exam may describe a downstream symptom and expect you to identify the upstream cause.

```
  ROOT CAUSE                    FIRST-ORDER EFFECT              SECOND-ORDER EFFECT
  ────────────────────────────────────────────────────────────────────────────────────

  Missing index          ──►    Table scan on every query  ──►  CPU spikes
                                                           ──►  Disk I/O saturates
                                                           ──►  Other queries queue
                                                           ──►  Throughput drops

  Table scan             ──►    Excessive lock acquisition ──►  Lock contention
                                                           ──►  Deadlocks
                                                           ──►  Transactions killed
                                                           ──►  Application errors

  Unmanaged txn log      ──►    Log fills disk             ──►  Writes blocked
                                                           ──►  Transactions fail
                                                           ──►  Application down
                                                           ──►  Data loss risk

  Connection pool        ──►    Max connections reached    ──►  New users refused
  exhaustion                                               ──►  Application errors
                                                           ──►  Perceived outage

  Fragmented indexes     ──►    Random I/O instead of      ──►  Disk latency rises
                                sequential reads           ──►  Buffer pool thrashes
                                                           ──►  Memory pressure
                                                           ──►  Page life drops
```

**The rule:** When the exam describes a symptom (CPU spike, slow queries, application errors), trace **backward** through this cascade to find the root cause. The answer is usually one or two levels upstream from the symptom described.

---

## 4. The Maintenance Toolkit

### Index Lifecycle — From Creation to Retirement

Indexes are not static. They have a lifecycle, and Domain 3 owns the operational phase:

```
  DOMAIN 1 (DESIGN)              DOMAIN 3 (OPERATE)

  CREATE INDEX              ──►  Monitor usage (is it being used?)
  (DDL decision)                      │
                                      ├── Used frequently?
                                      │     │
                                      │     ├── Fragmented > 30%? → REBUILD
                                      │     │   (drops and recreates;
                                      │     │    requires more resources;
                                      │     │    may need maintenance window)
                                      │     │
                                      │     └── Fragmented 5-30%? → REORGANIZE
                                      │         (in-place defrag;
                                      │          less resource-intensive;
                                      │          can run online)
                                      │
                                      ├── Not used at all?
                                      │     └── REMOVE IT
                                      │         Every unused index costs:
                                      │         • Write overhead on INSERT/UPDATE/DELETE
                                      │         • Storage space
                                      │         • Backup size increase
                                      │
                                      └── Query still slow despite index?
                                            └── Consider COVERING INDEX
                                                Include all columns the query
                                                needs → eliminates key lookups
                                                entirely
```

### Visual: Rebuild vs Reorganize — When to Use Which

```
  FRAGMENTATION LEVEL              ACTION                  CHARACTERISTICS

   0%  ████████████████████████     No action needed        Pages in order
       (sequential)

   5%  ████████░███████░███████     REORGANIZE              • In-place compaction
       (lightly scattered)                                  • Can run online
                                                            • Lower resource cost
                                                            • Good for routine
                                                              maintenance windows

  30%  ███░░████░░██░░███░░████     REBUILD                 • Drops and recreates
       (heavily scattered)                                  • Higher resource cost
                                                            • May require offline
                                                              or maintenance window
                                                            • Resets fragmentation
                                                              to ~0%

 >70%  █░░░██░░░█░░░██░░░█░░░██    REBUILD (urgent)        • Random I/O dominant
       (severely fragmented)                                • Performance visibly
                                                              degraded
                                                            • Every query pays
                                                              the penalty
```

### Patch Management — The Risk Gradient

```
  SECURITY PATCHES                MAINTENANCE PATCHES           UPDATES (Features)
  ┌────────────────────┐          ┌────────────────────┐        ┌────────────────────┐
  │ Close known        │          │ Bug fixes,          │        │ New features,      │
  │ vulnerabilities    │          │ stability           │        │ cumulative fixes   │
  │                    │          │                    │        │                    │
  │ URGENCY: HIGH      │          │ URGENCY: MODERATE   │        │ URGENCY: LOW       │
  │ Delay = exposure   │          │                    │        │ May introduce       │
  │                    │          │                    │        │ breaking changes    │
  │ Still test first!  │          │ Still test first!  │        │ Still test first!  │
  │ (non-production)   │          │ (non-production)   │        │ (non-production)   │
  └────────────────────┘          └────────────────────┘        └────────────────────┘

  ALL THREE follow the same rule:
  ┌──────────────────────────────────────────────────────────────────────┐
  │  Test in non-production → Validate → Apply to production via       │
  │  change management → Monitor for regressions → Document            │
  └──────────────────────────────────────────────────────────────────────┘

  NEVER apply directly to production. NEVER.
  Even "urgent" security patches are tested first.
  The only variable is how fast you move through the process.
```

### Integrity Checks — Detecting Corruption Before It Spreads

```
  DATABASE INTEGRITY CHECK PIPELINE

  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
  │ CONSISTENCY      │     │ DATA CORRUPTION  │     │ BACKUP           │
  │ CHECKS           │     │ CHECKS           │     │ VALIDATION       │
  │                  │     │                  │     │                  │
  │ DBCC CHECKDB     │     │ Checksums before │     │ Restore backup   │
  │ CHECK TABLE      │     │ and after maint  │     │ to test env      │
  │ pg_catalog       │     │ Row count verify │     │ Verify data      │
  │                  │     │ Hash comparison  │     │ integrity        │
  │ Detects:         │     │                  │     │                  │
  │ • Page-level     │────►│ Validates:       │────►│ Proves:          │
  │   corruption     │     │ • Data not       │     │ • Backup is      │
  │ • Structural     │     │   altered by     │     │   actually       │
  │   damage         │     │   maintenance    │     │   restorable     │
  │                  │     │ • Counts match   │     │ • No corruption  │
  │ Requires:        │     │   expectations   │     │   in backup file │
  │ Table locks      │     │                  │     │                  │
  │ (schedule during │     │                  │     │ False confidence  │
  │ maint window)    │     │                  │     │ killer: untested │
  └─────────────────┘     └─────────────────┘     │ backups          │
                                                   └─────────────────┘
```

---

## 5. Change Management — The Wrapper Process

Change management is not one process among many in Domain 3. It is the **meta-process that governs all other processes**. Every maintenance action, every schema change, every patch, every data management task must pass through it before touching production.

```
  ┌──────────────────────────────────────────────────────────────────────────┐
  │                       CHANGE MANAGEMENT PIPELINE                        │
  │                                                                          │
  │  ┌────────┐  ┌─────────┐  ┌────────────┐  ┌───────────┐  ┌──────────┐  │
  │  │  PLAN  │─►│ APPROVE │─►│ COMMUNICATE│─►│ IMPLEMENT │─►│ VALIDATE │  │
  │  │        │  │  (CAB)  │  │            │  │           │  │          │  │
  │  │ What   │  │         │  │ Who is     │  │ Execute   │  │ Did it   │  │
  │  │ will   │  │ Sign-off│  │ affected?  │  │ the       │  │ work?    │  │
  │  │ change?│  │ before  │  │ Downtime?  │  │ change    │  │ Test     │  │
  │  │ Why?   │  │ action  │  │ Rollback   │  │           │  │ results  │  │
  │  │ Risk?  │  │         │  │ criteria?  │  │           │  │ match    │  │
  │  │ Roll-  │  │         │  │            │  │           │  │ expected │  │
  │  │ back?  │  │         │  │            │  │           │  │ outcome? │  │
  │  └────────┘  └─────────┘  └────────────┘  └───────────┘  └────┬─────┘  │
  │                                                                │        │
  │                                                         ┌──────▼─────┐  │
  │                                                         │  DOCUMENT  │  │
  │                                                         │            │  │
  │                                                         │ What was   │  │
  │                                                         │ changed,   │  │
  │                                                         │ when, by   │  │
  │                                                         │ whom,      │  │
  │                                                         │ outcome    │  │
  │                                                         └────────────┘  │
  └──────────────────────────────────────────────────────────────────────────┘

  EXAM TRAPS:
  • "Implement, then get approval" → WRONG (approve before implement)
  • "Apply the fix, then communicate" → WRONG (communicate before implement)
  • "Document only if something went wrong" → WRONG (always document)
```

### What Flows Through Change Management

| Change type | Specific items | Change management applies? |
|---|---|---|
| **Optimization** | Index rebuild, query rewrite, stats update | Yes in production |
| **Patch management** | Security patches, updates, maintenance patches | Yes — test first, then apply via pipeline |
| **Schema changes** | ALTER TABLE, new indexes, new views | Yes — update data dictionary and ERD after |
| **Capacity** | Storage expansion, RAM upgrade, CPU addition | Yes — affects availability during implementation |
| **Upgrades** | Major DBMS version upgrade | Yes — requires compatibility testing, rollback plan |
| **Vulnerability remediation** | Fix identified security issues | Yes — within defined SLA |
| **Database refresh** | Copy production to lower environments | Yes — must mask sensitive data before copying |
| **Release schedules** | Application deployments touching schema | Yes — coordinated with development teams |

### Database Refresh — Where Domain 3 Meets Domain 4

```
  PRODUCTION                                    DEV / TEST
  ┌──────────────────┐                          ┌──────────────────┐
  │ Real data         │                          │ Masked data       │
  │ (PII, PHI, PCI)  │                          │ (safe to use)    │
  │                  │                          │                  │
  │ Customer: Lee    │──── copy ─────────┐      │ Customer: X39K   │
  │ SSN: 123-45-6789 │                   │      │ SSN: ***-**-**** │
  │ Card: 4111...    │                   │      │ Card: XXXX...    │
  └──────────────────┘                   │      └──────────────────┘
                                         │              ▲
                                         │              │
                                    ┌────▼──────────────┘
                                    │   MASKING
                                    │   (MUST happen BEFORE
                                    │    data reaches dev/test)
                                    │
                                    │   Static masking:
                                    │   Permanently alter the copy.
                                    │   Production unchanged.
                                    └───────────────────────────
```

Skipping masking during refresh = PII/PHI in dev = compliance violation. The exam tests this explicitly.

---

## 6. The Documentation Stack

Documentation in Domain 3 is not a list of document types. It's an interconnected system where each document serves a specific audience and a specific operational function.

```
  WHO NEEDS IT              WHAT THEY NEED              WHICH DOCUMENT

  New DBA joining team ──►  "How is the schema         ──► Data dictionary + ERD
                            structured?"

  DBA performing           "What are the exact         ──► SOPs + Maintenance docs
  routine maintenance ──►   steps for this task?"

  DBA making a change ──►  "What is the correct        ──► Change management records
                            process?"                       (within maintenance docs)

  Auditor (internal) ──►   "Prove your processes       ──► Organizational compliance
                            follow policy"                  documentation

  Auditor (external) ──►   "Prove you meet PCI DSS /   ──► Third-party compliance docs
                            GDPR / SOC 2 / HIPAA"          + audit log review results
                                                           + data dictionary (prove
                                                             encryption of sensitive cols)

  Future you (or your ──►  "What happened last time    ──► Maintenance documentation
   replacement)             we did this?"                   (runbooks, change history)
```

### Tools Matched to Documents

| Tool | Document it produces | Why this tool |
|---|---|---|
| **UML editors** (Lucidchart, draw.io, Visio) | ERDs, data flow diagrams | Visual diagramming with relationship notation |
| **Spreadsheet tools** (Excel, Sheets) | Data dictionaries, capacity plans, audit summaries | Tabular data with sorting, filtering, formulas |
| **Word processors** (Word, Google Docs) | SOPs, maintenance docs, compliance narratives | Long-form procedural text with formatting |

---

## 7. Data Management Tasks — Schema in Motion

Objective 3.4 is where Domain 3 loops back to Domain 1. Every task here uses SQL from Domain 1 but is governed by change management from Domain 3:

```
  DOMAIN 1 KNOWLEDGE                    DOMAIN 3 GOVERNANCE
  (what to do)                           (how to do it safely)

  CREATE TABLE ──────────────────────►  Change management pipeline
  ALTER TABLE ... ADD COLUMN ────────►  + Update data dictionary
  CREATE INDEX ──────────────────────►  + Update ERD if relationships change
  INSERT / UPDATE / DELETE ──────────►  + Validate after implementation
  CREATE VIEW ───────────────────────►  + Document what was done
  CREATE MATERIALIZED VIEW ──────────►
```

### View vs Materialized View — The Decision System

This decision isn't arbitrary. It follows directly from the workload:

```
  START: What is the query pattern?
    │
    ├── Query runs frequently AND is expensive (multi-table JOINs,
    │   aggregations, GROUP BY)?
    │     │
    │     ├── Can the data be slightly stale? (refreshed periodically)
    │     │     │
    │     │     YES ──► MATERIALIZED VIEW
    │     │             • Physically stores the result
    │     │             • Fast reads (pre-computed)
    │     │             • Must be refreshed (schedule it)
    │     │             • Uses storage
    │     │
    │     └── Must always be current? (real-time accuracy required)
    │           │
    │           YES ──► VIEW (but optimize underlying queries/indexes)
    │                   • Runs the SELECT every time
    │                   • Always current
    │                   • Can be slow if underlying query is complex
    │
    └── Query is simple, or runs infrequently?
          │
          YES ──► VIEW
                  • No storage overhead
                  • Simplifies access
                  • Restricts columns (security)
                  • Provides stable interface
```

---

## 8. The Deadlock — Understanding the Mechanism

Deadlocks are a specific exam topic that benefits from a visual understanding of the mechanism:

```
  NORMAL LOCKING (no problem):

  Transaction A:  Lock Table1 → Use Table1 → Release Table1 → Lock Table2 → ...
  Transaction B:  Waits...                                    Lock Table1 → ...

  No conflict — B waits until A is done with Table1.


  DEADLOCK (problem):

  Time ──────────────────────────────────────────────────────────────────►

  Transaction A:  Lock Table1 ────────────── wants Table2 ──► BLOCKED
                                                    ▲              │
                                                    │              │
                                              neither can          │
                                              proceed              │
                                                    │              │
                                                    ▼              │
  Transaction B:  Lock Table2 ────────────── wants Table1 ──► BLOCKED


  RESOLUTION: DBMS detects the cycle, kills one transaction (the "victim"),
              rolls it back, and lets the other proceed.

  WHAT THE DBA DOES:
  1. Monitor deadlock frequency and objects involved
  2. If recurring: redesign transaction order so both access tables
     in the SAME sequence (A→B, not A→B + B→A)
  3. Shorten transactions (less time holding locks)
  4. Add appropriate indexes (reduces lock escalation)
```

---

## 9. Domain 3 Decision Engine

### Filter 1: Where on the operations loop?

| Question is about... | Loop position | Sub-objective |
|---|---|---|
| Metrics, alerts, baselines, logs, connections, replication | MONITOR | 3.1 |
| Optimization, patching, integrity checks, change management | MAINTAIN | 3.2 |
| Data dictionaries, ERDs, SOPs, compliance docs, tools | DOCUMENT | 3.3 |
| Creating tables/indexes/views, modifying data, relationships | DATA TASKS | 3.4 |

### Filter 2: Which clock?

| Continuous (real-time) | Periodic (scheduled) |
|---|---|
| CPU/memory/disk monitoring | Index rebuild/reorganize |
| Replication lag tracking | Patch application |
| Connection monitoring | DBCC CHECKDB / integrity checks |
| Deadlock detection | Audit log review |
| Transaction log growth alerts | Capacity planning review |
| Backup job success/failure alerts | Database refresh |

### Filter 3: Is it a boundary question?

| Pair | Distinguishing property |
|---|---|
| Baseline vs Trending | Baseline = snapshot of "normal." Trending = baseline compared over time |
| Rebuild vs Reorganize | Rebuild = drop/recreate (heavy, >30%). Reorganize = in-place compaction (light, 5-30%) |
| View vs Materialized view | View = no data stored, always current. Materialized = stored snapshot, stale until refreshed |
| Transaction log vs System log | Transaction = data changes for recovery. System = errors and events for troubleshooting |
| Audit log vs System log | Audit = who did what (compliance). System = what happened (operations) |
| Query optimization vs Index optimization | Query = rewrite the SQL. Index = change the data structure the SQL uses |
| Intentional vs Unintentional redundancy | Intentional = denormalization (design choice). Unintentional = duplication (bug) |
| Data dictionary vs ERD | Data dictionary = text catalog (columns, types, constraints). ERD = visual diagram (entities, relationships) |
| SOP vs Maintenance doc | SOP = step-by-step procedure for a task. Maintenance doc = schedules, runbooks, change history |
| Organizational vs Third-party compliance | Organizational = internal policies. Third-party = external standards (PCI, SOC 2, GDPR) |
| Security patch vs Maintenance patch | Security = close vulnerabilities (urgent). Maintenance = bug fixes (routine) |

### Filter 4: Is it a sequence question?

| Process | Correct order | The exam trap |
|---|---|---|
| Change management | Plan → Approve → Communicate → Implement → Validate → Document | Implementing before approval. Skipping communication. Skipping documentation. |
| Performance diagnosis | Check baseline → Analyze execution plan → Identify root cause → Fix | Jumping to hardware upgrade. Fixing symptoms instead of causes. |
| Patch application | Test in non-production → Validate → Apply to production → Monitor → Document | Applying directly to production. |
| Database refresh | Identify source data → Mask sensitive data → Copy to target → Validate | Copying before masking. |

### Filter 5: Is it a "what should the DBA investigate?" question?

| Symptom described | Investigate | Why |
|---|---|---|
| CPU spike | Compare to baseline first | May be normal |
| Slow queries | Execution plan (table scans, key lookups, sort spills) | Evidence-based diagnosis |
| Backup file size anomaly | Compare to previous backup sizes | Sudden change = potential corruption or data loss |
| Spike in failed connections | Brute-force attack OR stale application credentials | Two very different root causes |
| Recurring deadlocks | Transaction order and lock objects | Indicates design issue |
| Declining throughput | Baseline comparison → index fragmentation → lock contention | Work through the cascade |
| Transaction log filling disk | Log backup schedule | Unmanaged logs = ticking bomb |

---

## 10. Domain 3 Ripple Effects

| If this happens... | Cascade through the system... |
|---|---|
| **No baseline established** | → Can't set meaningful thresholds → Alerts don't fire or fire constantly (noise) → Real problems missed → Silent degradation → Outage |
| **Index rebuild skipped for months** | → Fragmentation climbs → Random I/O increases → Disk latency rises → Buffer pool thrashes → CPU compensates → Throughput drops → Users notice → Emergency maintenance during business hours |
| **Transaction log not backed up** | → Log grows continuously → Fills disk → ALL writes stop → Database effectively down → Point-in-time recovery impossible → RPO violated |
| **Change applied without approval** | → Breaks production → No rollback plan documented → Extended outage → Compliance violation (change management process not followed) → Audit finding |
| **Data dictionary not updated after schema change** | → Other DBAs reference stale info → Queries built on wrong assumptions → Bug reports from applications → Hours spent debugging → Trust in documentation erodes → Team stops using data dictionary → Compound drift |
| **Database refresh without masking** | → Real PII/PHI in dev environment → Developer accidentally exposes data → Compliance violation (PCI DSS, GDPR, HIPAA) → Breach notification required → Legal consequences |
| **Audit logs not reviewed periodically** | → Privilege escalation goes undetected → Unauthorized schema changes unnoticed → Compliance auditor requests evidence → None available → Audit failure |
| **Dead index never removed** | → Every INSERT/UPDATE/DELETE pays overhead → Write performance degrades gradually → Backup size grows → Storage consumption increases → But nobody notices because the degradation is slow (boiling frog) |
| **Deadlocks ignored** | → Recurring victim transactions → Application retries increase load → Users experience intermittent failures → Trust in system erodes → Eventually escalates to outage during peak load |
