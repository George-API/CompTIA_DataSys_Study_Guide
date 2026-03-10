# Domain 5 — Business Continuity: The Survival System

Domain 5 is not two separate objectives about disaster recovery and backups. It is one **survival system** — the last line of defense that exists because every control in every other domain can fail. Hardware dies. Ransomware encrypts. Floods destroy data centers. Humans make mistakes. Domain 5 answers the question every other domain hopes to avoid: "When everything else fails, can we recover?"

Every decision in this domain is governed by exactly two numbers: **RPO** (how much data can we lose) and **RTO** (how fast must we recover). These two numbers are the master control panel. They determine your DR technique, your backup strategy, your storage architecture, your budget, and your testing requirements. If you understand how RPO and RTO drive decisions, you can derive the answer to any Domain 5 question.

---

## 1. The Survival System

Domain 5 operates as three layers of protection. Each layer catches what the layer above it missed. The exam tests whether you understand which layer applies to a given scenario.

```
  ┌──────────────────────────────────────────────────────────────────────────┐
  │  LAYER 1: KEEP IT RUNNING (Domains 1-4)                                │
  │                                                                          │
  │  Monitoring, patching, security, access control, good design.           │
  │  Goal: prevent disasters from happening in the first place.            │
  │  This is not Domain 5 — but Domain 5 exists because this layer         │
  │  is never 100% effective.                                              │
  ├──────────────────────────────────────────────────────────────────────────┤
  │  LAYER 2: STAY ALIVE (DR Techniques — 5.1)                            │
  │                                                                          │
  │  Replication, mirroring, HA clustering, log shipping.                  │
  │  Goal: if the primary fails, switch to a standby INSTANTLY             │
  │  (or within minutes) with minimal data loss.                           │
  │                                                                          │
  │  This layer provides CONTINUITY — the system keeps running,            │
  │  possibly in a degraded state, while the primary is restored.          │
  ├──────────────────────────────────────────────────────────────────────────┤
  │  LAYER 3: REBUILD FROM SCRATCH (Backups — 5.2)                        │
  │                                                                          │
  │  Full, differential, incremental backups. Database dumps.              │
  │  Goal: if Layer 2 also fails (both primary AND standby are             │
  │  destroyed, or ransomware encrypts both), reconstruct the              │
  │  database from stored copies.                                          │
  │                                                                          │
  │  This is the LAST RESORT. Slowest recovery, highest data loss,        │
  │  but it's the only layer that survives total infrastructure            │
  │  destruction — if the backups are off-site and tested.                 │
  └──────────────────────────────────────────────────────────────────────────┘

  The exam asks: "Given this scenario, which layer is relevant?"

  • Primary server's CPU is spiking → Layer 1 (monitoring/optimization — D3)
  • Primary server crashes, standby exists → Layer 2 (failover — D5)
  • Both primary and standby destroyed → Layer 3 (restore from backup — D5)
  • Ransomware encrypts live DB and online backups → Layer 3
    (offline/immutable backups — the ONLY defense)
```

---

## 2. Governing Principles of Business Continuity

### Principle 1: RPO and RTO Are the Two Master Variables

Every other decision in Domain 5 is a function of these two numbers:

```
                              DISASTER
                              STRIKES
                                 │
     ◄─────── RPO ──────►       │       ◄──────── RTO ────────►
     (data loss tolerance)       │       (downtime tolerance)
                                 │
  ───────────────────────────────┼───────────────────────────────────► time
                                 │
  Last good              Data since       System         Users
  backup or              that point       restored       reconnected
  replication            IS LOST          and running    and working
  checkpoint
                                 │
  ┌──────────────────────────────┼──────────────────────────────────┐
  │                              │                                  │
  │  RPO drives:                 │  RTO drives:                    │
  │                              │                                  │
  │  • Backup FREQUENCY          │  • DR TECHNIQUE choice          │
  │    (RPO 1hr = backup         │    (RTO ~0 = HA clustering      │
  │     every hour minimum)      │     RTO hrs = cold restore)     │
  │                              │                                  │
  │  • REPLICATION type          │  • BUDGET                       │
  │    (RPO ~0 = synchronous)    │    (lower RTO = higher cost)    │
  │                              │                                  │
  │  • LOG SHIPPING interval     │  • AUTOMATION level             │
  │    (RPO = gap between        │    (lower RTO = more automated  │
  │     log shipments)           │     failover)                   │
  │                              │                                  │
  └──────────────────────────────┼──────────────────────────────────┘
                                 │
  RPO answers: "How much data    │  RTO answers: "How long can the
  can the business afford        │  business survive without the
  to LOSE?"                      │  database?"
```

**Exam implication:** When a question specifies an RPO or RTO, the answer is the technique or schedule that meets that number. RPO of 15 minutes + daily backups = violation. RTO of seconds + cold tape restore = violation.

### Principle 2: A Backup You've Never Tested Is a Hope, Not a Backup

The exam tests this principle directly. A backup file exists on disk, but:
- Has it ever been restored successfully?
- Has its hash been validated?
- Is the encryption key available?
- Is it stored where the same disaster can destroy it?

If the answer to any of these is "no," the backup is unreliable.

```
  THE BACKUP TRUST CHAIN — every link must hold

  Backup          Backup         Backup         Encryption     Backup
  was              completed      hash           key            was restored
  SCHEDULED        SUCCESSFULLY   VALIDATES      is             to a test env
  ───────────►    ───────────►   ───────────►  ───────────►   ───────────►
                                                                  │
  If any link breaks:                                             │
  ┌──────────────────────────────────────────────────────────┐    │
  │ Not scheduled     → no backup exists                     │    │
  │ Failed silently   → file is 0 bytes or missing           │    │
  │ Hash mismatch     → file is corrupt or tampered          │    │
  │ Key lost          → encrypted backup is unreadable       │    │
  │ Never tested      → may fail when actually needed        │    │
  └──────────────────────────────────────────────────────────┘    │
                                                                  │
  Only if ALL links hold ─────────────────────────────────────────┘
  is the backup trustworthy.
```

### Principle 3: The Cost-Protection Trade-Off Is a Spectrum

More protection costs more money and more complexity. The business requirement (RPO/RTO) determines where on the spectrum you must be — spending more than necessary wastes money, spending less than necessary violates the recovery objectives.

```
  COST & COMPLEXITY ────────────────────────────────────────────► HIGH

  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
  │ LOG SHIPPING  │  │ ASYNC        │  │ SYNC         │  │ HA           │
  │              │  │ REPLICATION  │  │ REPLICATION/ │  │ CLUSTERING   │
  │              │  │              │  │ MIRRORING    │  │              │
  │ RPO: hours   │  │ RPO: seconds │  │ RPO: ~zero   │  │ RPO: ~zero   │
  │ RTO: hours   │  │ RTO: minutes │  │ RTO: seconds │  │ RTO: ~zero   │
  │              │  │              │  │              │  │              │
  │ Ships txn    │  │ Continuous   │  │ Primary waits│  │ Multiple     │
  │ log backups  │  │ copy, but    │  │ for secondary│  │ nodes share  │
  │ on a         │  │ doesn't wait │  │ to ACK every │  │ workload;    │
  │ schedule     │  │ for ACK      │  │ write        │  │ auto-failover│
  │              │  │              │  │              │  │              │
  │ Cheapest     │  │ Good balance │  │ Write        │  │ Most $ $ $   │
  │ Most data    │  │ of cost and  │  │ latency cost │  │ Most complex │
  │ loss risk    │  │ protection   │  │ Safety first │  │ Near-zero    │
  │              │  │              │  │              │  │ downtime     │
  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘

  ◄── "We can tolerate          "We cannot tolerate ──►
       some data loss             ANY downtime or
       and some downtime"         data loss"
```

### Principle 4: Disaster Recovery Is a Cycle, Not an Event

The exam tests the full cycle — not just failover, but failback, documentation, and improvement:

```
  NORMAL           DISASTER          FAILOVER         STABILIZE
  OPERATIONS  ──►  OCCURS    ──►     TO DR SITE  ──►  ON DR SITE
      ▲                                                    │
      │                                                    │
      │            DOCUMENT          MONITOR           FAILBACK
      │            LESSONS   ◄──    STABILITY   ◄──   TO PRIMARY
      │            LEARNED                                 │
      │                │                                   │
      └────────────────┘                                   │
       Update DR plan                                      │
       with findings             Resync data ◄─────────────┘
                                 Validate integrity
                                 Switch traffic
```

Failback is not "just switch back." It's a six-step process with its own sequence and its own risks.

### Principle 5: Off-Site Saves You From the Disaster That Destroys Your Building

On-site backups protect against hardware failure, accidental deletion, and corruption. Off-site backups protect against everything on-site backups protect against **plus** the disasters that destroy the physical location: fire, flood, earthquake, theft.

```
  ON-SITE ONLY                           ON-SITE + OFF-SITE (3-2-1)
  ┌────────────────────────┐             ┌────────────────────────┐
  │ Fire destroys          │             │ Fire destroys          │
  │ server room            │             │ server room            │
  │         │              │             │         │              │
  │         ▼              │             │         ▼              │
  │ Primary DB: GONE       │             │ Primary DB: GONE       │
  │ Backup files: GONE     │             │ On-site backups: GONE  │
  │                        │             │                        │
  │ Recovery: IMPOSSIBLE   │             │ Off-site backup: SAFE  │
  │                        │             │                        │
  │                        │             │ Recovery: POSSIBLE     │
  │                        │             │ (transfer from off-site│
  │                        │             │  + restore)            │
  └────────────────────────┘             └────────────────────────┘

  THE 3-2-1 RULE:
  ┌───────────────────────────────────────────────────────────┐
  │  3 copies of data     (primary + 2 backups)              │
  │  2 different media    (disk + tape, disk + cloud)        │
  │  1 off-site           (survives local disaster)          │
  │                                                           │
  │  Protects against:                                        │
  │  • Single hardware failure (multiple copies)             │
  │  • Single media failure (different media types)          │
  │  • Site-level disaster (geographic separation)           │
  └───────────────────────────────────────────────────────────┘
```

### Principle 6: Ransomware Changed the Backup Equation

Traditional backup strategy assumed backups were safe once created. Ransomware changes this — it specifically targets backup files alongside live data. The only defense is backups that ransomware **cannot reach**:

```
  TRADITIONAL THREAT MODEL:

  Disaster destroys PRIMARY ──► Restore from BACKUP
  (hardware failure, flood)     (backup is safe because it's a separate file)


  RANSOMWARE THREAT MODEL:

  Ransomware encrypts PRIMARY ──► Also encrypts BACKUP files
  (if backups are on the         (if they're accessible from
   same network or online)        the compromised system)

       │
       ▼
  BOTH primary and backup are unreadable.
  Recovery: IMPOSSIBLE (unless you pay — and even then, not guaranteed)


  DEFENSE:

  ┌──────────────────────────────────────────────────────────────┐
  │  OFFLINE BACKUPS     Cannot be reached by ransomware         │
  │  (air-gapped)        because there is no network connection  │
  │                      to the backup storage.                  │
  │                                                              │
  │  IMMUTABLE BACKUPS   Cannot be modified or deleted even      │
  │  (WORM storage)      by administrators. Write once,          │
  │                      read many. Even if compromised,         │
  │                      the backup file cannot be encrypted     │
  │                      or overwritten.                         │
  └──────────────────────────────────────────────────────────────┘

  This is why "offline/immutable backups" is the PRIMARY mitigation
  for ransomware on the exam — not antivirus, not patching, not
  firewalls (those are secondary layers).
```

---

## 3. The DR Technique System

The four DR techniques are not a list to memorize. They form a system where each technique occupies a specific position on the cost-protection spectrum, and each is suited to a specific RPO/RTO requirement.

```
  ┌──────────────────────────────────────────────────────────────────────────┐
  │                     DR TECHNIQUE ARCHITECTURE                            │
  │                                                                          │
  │  LOG SHIPPING                                                           │
  │  ┌─────────────────────────────────────────────────────────────────┐    │
  │  │                                                                 │    │
  │  │  PRIMARY ────── txn log backup ────► STANDBY                   │    │
  │  │                 (every N minutes)     (restores logs)           │    │
  │  │                                                                 │    │
  │  │  Mechanism: Ship log files on a schedule. Standby replays them.│    │
  │  │  Gap: data generated BETWEEN shipments is at risk.             │    │
  │  │  Standby is always N minutes behind.                           │    │
  │  │                                                                 │    │
  │  │  When to use: Lowest budget. Acceptable RPO of minutes-hours.  │    │
  │  └─────────────────────────────────────────────────────────────────┘    │
  │                                                                          │
  │  ASYNCHRONOUS REPLICATION / MIRRORING                                   │
  │  ┌─────────────────────────────────────────────────────────────────┐    │
  │  │                                                                 │    │
  │  │  PRIMARY ════ continuous stream ═══► STANDBY                   │    │
  │  │          (doesn't wait for ACK)                                │    │
  │  │                                                                 │    │
  │  │  Mechanism: Every change streams to standby immediately,       │    │
  │  │  but PRIMARY commits without waiting for standby confirmation. │    │
  │  │  Gap: last few seconds/transactions may not have reached       │    │
  │  │  standby when primary dies.                                    │    │
  │  │                                                                 │    │
  │  │  When to use: Need low RPO but can't afford write latency.    │    │
  │  └─────────────────────────────────────────────────────────────────┘    │
  │                                                                          │
  │  SYNCHRONOUS REPLICATION / MIRRORING                                    │
  │  ┌─────────────────────────────────────────────────────────────────┐    │
  │  │                                                                 │    │
  │  │  PRIMARY ════ write ═══► STANDBY ═══► ACK ═══► PRIMARY commits │    │
  │  │          (waits for confirmation before committing)            │    │
  │  │                                                                 │    │
  │  │  Mechanism: Primary does NOT commit until standby confirms     │    │
  │  │  it received the data. Zero gap.                               │    │
  │  │  Cost: every write is slower (round-trip to standby + ACK).   │    │
  │  │                                                                 │    │
  │  │  When to use: RPO must be near-zero. Accept write latency.    │    │
  │  └─────────────────────────────────────────────────────────────────┘    │
  │                                                                          │
  │  HIGH AVAILABILITY (HA) CLUSTERING                                      │
  │  ┌─────────────────────────────────────────────────────────────────┐    │
  │  │                                                                 │    │
  │  │      ┌──────┐  ┌──────┐  ┌──────┐                              │    │
  │  │      │Node 1│  │Node 2│  │Node 3│  ← shared workload          │    │
  │  │      └──┬───┘  └──┬───┘  └──┬───┘                              │    │
  │  │         └─────────┼─────────┘                                   │    │
  │  │                   │                                              │    │
  │  │          Shared storage / sync replication                      │    │
  │  │                                                                 │    │
  │  │  Active-passive: standby nodes idle until needed.              │    │
  │  │  Active-active: all nodes serve traffic simultaneously.        │    │
  │  │                                                                 │    │
  │  │  Mechanism: If any node fails, others absorb its workload     │    │
  │  │  automatically. No manual intervention.                        │    │
  │  │  Cost: most expensive and most complex to configure.          │    │
  │  │                                                                 │    │
  │  │  When to use: RTO and RPO must both be near-zero.             │    │
  │  │  Includes: failover groups, always-on configurations.         │    │
  │  └─────────────────────────────────────────────────────────────────┘    │
  └──────────────────────────────────────────────────────────────────────────┘
```

### The Synchronous vs Asynchronous Decision

This trade-off appears repeatedly on the exam. One rule governs it:

```
  THE QUESTION PRIORITIZES...         CHOOSE...

  "Data safety"                  ──►  SYNCHRONOUS
  "Zero data loss"                    (primary waits for ACK)
  "No acceptable data loss"           RPO: ~zero
  "Financial transactions"            Cost: write latency on every transaction

  "Performance"                  ──►  ASYNCHRONOUS
  "Minimize latency"                  (primary commits immediately)
  "High write throughput"             RPO: seconds-minutes
  "Geographic distance"               Cost: risk of losing recent transactions
  (latency makes sync impractical)
```

---

## 4. The Backup System

### Backup Types — How They Relate Over Time

The three backup types are not independent choices. They form a **strategy** — a combination designed to balance storage cost, backup speed, and restore complexity.

```
  A TYPICAL WEEK:

  Sun        Mon        Tue        Wed        Thu        Fri        Sat
  FULL       Diff/Inc   Diff/Inc   Diff/Inc   Diff/Inc   Diff/Inc   Diff/Inc


  WITH DIFFERENTIAL:
  ┌────┐     ┌────┐     ┌────┐     ┌────┐     ┌────┐     ┌────┐     ┌────┐
  │FULL│     │Diff│     │Diff│     │Diff│     │Diff│     │Diff│     │Diff│
  │████│     │░   │     │░░  │     │░░░ │     │░░░░│     │░░░░░│    │░░░░░░│
  └────┘     └────┘     └────┘     └────┘     └────┘     └────┘     └────┘
  All data   Since Sun  Since Sun  Since Sun  Since Sun  Since Sun  Since Sun
                        ◄── each GROWS (cumulative since last full) ──►

  To restore Thursday:  FULL (Sun) + DIFF (Thu) = 2 files
  Restore: FAST (only 2 files)   Storage: MODERATE (diffs grow)


  WITH INCREMENTAL:
  ┌────┐     ┌────┐     ┌────┐     ┌────┐     ┌────┐     ┌────┐     ┌────┐
  │FULL│     │Inc │     │Inc │     │Inc │     │Inc │     │Inc │     │Inc │
  │████│     │░   │     │░   │     │░   │     │░   │     │░   │     │░   │
  └────┘     └────┘     └────┘     └────┘     └────┘     └────┘     └────┘
  All data   Since Sun  Since Mon  Since Tue  Since Wed  Since Thu  Since Fri
             ◄── each stays SMALL (only since last backup) ──►

  To restore Thursday:  FULL (Sun) + INC (Mon) + INC (Tue) + INC (Wed)
                        + INC (Thu) = 5 files, IN ORDER
  Restore: SLOW (5 files in sequence)   Storage: SMALL (each inc is tiny)
```

### The Backup Strategy Decision

```
  START: What does the business need?
    │
    ├── Fastest possible restore, budget available for storage?
    │     └── FULL backups more frequently
    │         Simplest restore (1 file). Largest storage.
    │
    ├── Balance of restore speed and storage?
    │     └── FULL (weekly) + DIFFERENTIAL (daily)
    │         Restore: 2 files. Diffs grow over the week
    │         but reset when next full runs.
    │
    └── Minimal storage, backup window is tight?
          └── FULL (weekly) + INCREMENTAL (daily)
              Smallest daily files. But restore requires
              EVERY incremental in sequence. Slowest restore.
              Any missing/corrupt incremental = broken chain.
```

### Database Dumping — The Portable Alternative

```
  PHYSICAL BACKUP                           LOGICAL BACKUP (DUMP)
  (file-level copy)                          (pg_dump, mysqldump, expdp)
  ┌──────────────────────────┐              ┌──────────────────────────┐
  │ Copies raw database       │              │ Exports objects and data │
  │ files (data + log)        │              │ as SQL scripts or        │
  │                           │              │ portable format          │
  │ Fast for large DBs        │              │                          │
  │ Platform-specific         │              │ Slower for large DBs     │
  │ (same DBMS version)       │              │ Cross-platform portable  │
  │                           │              │                          │
  │ Used for: routine         │              │ Used for: migrations,    │
  │ backups, DR               │              │ selective restores,      │
  │                           │              │ moving to different      │
  │                           │              │ DBMS or version          │
  └──────────────────────────┘              └──────────────────────────┘
```

### The Complete Backup Lifecycle

Every backup follows the same lifecycle from creation to disposal:

```
  CREATE            VERIFY             STORE              RETAIN           DISPOSE
  ────────          ──────             ─────              ──────           ───────

  Schedule          Generate hash      On-site            Retention        Purge
  and automate ──►  (SHA-256) at  ──►  (fast restore) ──► policy     ──►  (delete after
  (cron, SQL        creation time      +                  defines         period expires)
  Agent)            for later          Off-site           how long        OR
                    validation         (disaster                          Archive
                                       protection)                       (move to cold
                    Test restore       3-2-1 rule                         storage for
                    periodically                                          legal/regulatory
                                       Encrypt backups                    holds)
                                       Retain keys for
                                       same duration                      Ensure archived
                                       as backup                          backups are
                                                                          encrypted +
                                                                          keys retained
```

---

## 5. The Failover-Failback Cycle

Disaster recovery is not "fail over and you're done." It's a **complete cycle** that ends with returning to normal operations and updating the DR plan with lessons learned. The exam tests the failback sequence explicitly.

```
  ┌─────────────────────────────────────────────────────────────────────────┐
  │                    THE COMPLETE DR CYCLE                                 │
  │                                                                         │
  │  PHASE 1: DISASTER OCCURS                                              │
  │  ┌─────────────────────────────────────────────────────────────────┐   │
  │  │ Primary fails (hardware, ransomware, natural disaster, etc.)   │   │
  │  │ Detection: monitoring alerts fire (D3) or manual discovery     │   │
  │  │ Decision: activate DR plan                                     │   │
  │  └──────────────────────────────┬──────────────────────────────────┘   │
  │                                 ▼                                       │
  │  PHASE 2: FAILOVER                                                     │
  │  ┌─────────────────────────────────────────────────────────────────┐   │
  │  │ Switch operations to DR site / standby server                  │   │
  │  │ • Automatic (HA clustering, always-on) or manual               │   │
  │  │ • DNS update, connection string change, or load balancer switch│   │
  │  │ • Users reconnect to standby                                   │   │
  │  └──────────────────────────────┬──────────────────────────────────┘   │
  │                                 ▼                                       │
  │  PHASE 3: OPERATE ON DR SITE                                          │
  │  ┌─────────────────────────────────────────────────────────────────┐   │
  │  │ System runs on standby. May be degraded (read-only replicas,  │   │
  │  │ reduced capacity, higher latency if geographically distant).  │   │
  │  │ Simultaneously: begin restoring primary environment.           │   │
  │  └──────────────────────────────┬──────────────────────────────────┘   │
  │                                 ▼                                       │
  │  PHASE 4: FAILBACK (exam-tested sequence)                             │
  │  ┌─────────────────────────────────────────────────────────────────┐   │
  │  │                                                                 │   │
  │  │  Step 1: Verify primary environment is RESTORED and HEALTHY   │   │
  │  │          (hardware replaced, OS installed, DBMS configured)    │   │
  │  │                                                                 │   │
  │  │  Step 2: RESYNCHRONIZE data from DR site to primary           │   │
  │  │          (DR site has the most recent data now)                │   │
  │  │                                                                 │   │
  │  │  Step 3: VALIDATE data integrity                              │   │
  │  │          • Row counts match                                    │   │
  │  │          • Checksums / hash comparison                         │   │
  │  │          • Referential integrity confirmed                    │   │
  │  │                                                                 │   │
  │  │  Step 4: SWITCH traffic back to primary                       │   │
  │  │          • Update DNS records                                  │   │
  │  │          • Update connection strings                           │   │
  │  │          • Reconfigure load balancer                           │   │
  │  │                                                                 │   │
  │  │  Step 5: MONITOR closely for a defined stability period       │   │
  │  │                                                                 │   │
  │  │  Step 6: UPDATE DR documentation with lessons learned         │   │
  │  │                                                                 │   │
  │  └─────────────────────────────────────────────────────────────────┘   │
  │                                                                         │
  │  EXAM TRAPS:                                                            │
  │  • "Switch traffic before resyncing data" → WRONG (data will diverge) │
  │  • "Skip validation, just switch back" → WRONG (integrity unknown)    │
  │  • "Skip documentation" → WRONG (lessons lost, same mistakes repeat)  │
  │                                                                         │
  └─────────────────────────────────────────────────────────────────────────┘
```

---

## 6. DR Documentation — The Operational Memory

DR documentation is not paperwork. It's the instruction manual that enables recovery when the people who built the system may not be available (they could be on vacation, unreachable, or no longer employed). Each document type serves a specific recovery function.

```
  DOCUMENT                    ANSWERS THE QUESTION              USED WHEN

  ┌──────────────────┐        ┌──────────────────────────┐     ┌──────────────┐
  │ MANUALS          │        │ "What are the exact      │     │ During       │
  │                  │───────►│  step-by-step recovery   │────►│ failover and │
  │                  │        │  procedures?"            │     │ failback     │
  └──────────────────┘        └──────────────────────────┘     └──────────────┘

  ┌──────────────────┐        ┌──────────────────────────┐     ┌──────────────┐
  │ SYSTEM SECURITY  │        │ "What security controls  │     │ During       │
  │ PLAN             │───────►│  must be maintained      │────►│ rebuild and  │
  │                  │        │  during and after DR?"   │     │ compliance   │
  └──────────────────┘        └──────────────────────────┘     │ audit        │
                                                               └──────────────┘

  ┌──────────────────┐        ┌──────────────────────────┐     ┌──────────────┐
  │ COOP             │        │ "How does the broader    │     │ During       │
  │ (Continuity of   │───────►│  organization maintain   │────►│ extended     │
  │  Operations Plan)│        │  essential functions?"   │     │ outage       │
  └──────────────────┘        └──────────────────────────┘     └──────────────┘

  ┌──────────────────┐        ┌──────────────────────────┐     ┌──────────────┐
  │ BUILD            │        │ "How do we recreate the  │     │ When primary │
  │ DOCUMENTATION    │───────►│  entire server from      │────►│ is totally   │
  │                  │        │  scratch?"               │     │ destroyed    │
  │                  │        │  OS → DBMS → schema →    │     │              │
  │                  │        │  data restore            │     │              │
  └──────────────────┘        └──────────────────────────┘     └──────────────┘
```

### DR Plan Testing — Three Levels of Confidence

```
  LOWEST COST / RISK                                    HIGHEST CONFIDENCE
  ──────────────────────────────────────────────────────────────────────────

  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
  │ TABLETOP         │  │ SIMULATION       │  │ FULL FAILOVER    │
  │ EXERCISE         │  │                  │  │ TEST             │
  │                  │  │                  │  │                  │
  │ Walk through     │  │ Simulate failure │  │ Actually fail    │
  │ the plan on      │  │ in a TEST        │  │ over to DR site  │
  │ paper.           │  │ environment.     │  │ in production.   │
  │                  │  │                  │  │                  │
  │ Catches:         │  │ Catches:         │  │ Catches:         │
  │ • Procedural     │  │ • Technical      │  │ • Everything     │
  │   gaps           │  │   failures       │  │   the other      │
  │ • Missing roles  │  │ • Missing        │  │   tests miss     │
  │ • Unclear steps  │  │   scripts        │  │                  │
  │                  │  │ • Timing issues  │  │ Risk:            │
  │ Cost: minimal    │  │                  │  │ • Actual outage  │
  │ Risk: none       │  │ Cost: moderate   │  │   if it fails    │
  │                  │  │ Risk: low        │  │ • Highest cost   │
  │                  │  │                  │  │                  │
  │ Frequency:       │  │ Frequency:       │  │ Frequency:       │
  │ quarterly        │  │ semi-annually    │  │ annually         │
  └──────────────────┘  └──────────────────┘  └──────────────────┘
```

---

## 7. Retention — The End-of-Life Decision

Retention policy determines what happens to backups after their useful life. It connects Domain 5 directly to Domain 4 (governance and compliance).

```
  BACKUP IS CREATED
       │
       ▼
  ACTIVELY USEFUL (within restore window)
       │
       │  Retention period governed by:
       │  • Business policy (how long do we need this data?)
       │  • Regulatory requirements (PCI DSS, GDPR, HIPAA mandates)
       │
       ▼
  RETENTION PERIOD EXPIRES
       │
       ├── No regulatory or legal hold?
       │     └── PURGE
       │         Delete the backup file.
       │         Routine operational cleanup.
       │         Reclaim storage.
       │
       └── Regulatory or legal hold exists?
             └── ARCHIVE
                 Move to long-term, low-cost storage:
                 tape, cold cloud (Glacier), WORM storage.
                 │
                 ├── Encrypt the archive
                 ├── Retain encryption keys for the SAME duration
                 │   (key lost = data lost = compliance violation)
                 └── Document the retention period and review date

  RISK MAP:
  ┌────────────────────────────────────────────────────────────────┐
  │ Over-retention  → More data to protect → larger breach surface│
  │                 → Higher storage costs                        │
  │                 → More data subject to legal discovery        │
  │                                                                │
  │ Under-retention → Compliance violation                        │
  │                 → Can't recover from historical corruption    │
  │                 → Legal consequences if data was required     │
  └────────────────────────────────────────────────────────────────┘
```

---

## 8. Where Domain 5 Touches Every Other Domain

Business continuity is not isolated. It depends on and is depended upon by every other domain:

```
  DOMAIN 1 (DESIGN) ──────────────────────────── DOMAIN 5 (SURVIVE)
  Schema design determines                      what needs to be backed up.
  ACID guarantees (D1) ensure                    transactions are recoverable.
  Connection pooling (D1) affects                failover reconnection behavior.

  DOMAIN 2 (BUILD) ───────────────────────────── DOMAIN 5 (SURVIVE)
  DNS TTL (D2) determines                        failover speed.
  Build documentation (D2) enables               rebuild from scratch.
  Static IPs (D2) make                           firewall rules predictable
                                                 during failover.

  DOMAIN 3 (OPERATE) ─────────────────────────── DOMAIN 5 (SURVIVE)
  Monitoring (D3) detects                        the disaster.
  Backup alerts (D3) confirm                     backups completed.
  Transaction log management (D3) enables        point-in-time recovery.
  Replication lag monitoring (D3) validates       DR readiness.
  Change management (D3) governs                 DR plan updates.

  DOMAIN 4 (PROTECT) ─────────────────────────── DOMAIN 5 (SURVIVE)
  Encryption (D4) must extend to                 backup files.
  Encryption keys (D4) must be retained          as long as backups.
  Ransomware defense (D4) requires               offline/immutable backups.
  Compliance (D4) defines                        retention periods.
  Data masking (D4) must happen before           database refresh from
                                                 backup to lower env.
```

---

## 9. Domain 5 Decision Engine

### Filter 1: Which survival layer?

| Question is about... | Layer | Sub-objective |
|---|---|---|
| DR planning, documentation, techniques, RPO/RTO, failover/failback, testing | Stay alive (DR) | 5.1 |
| Backup types, scheduling, hash validation, storage, retention, dumping | Rebuild from scratch (Backup) | 5.2 |

### Filter 2: Is it an RPO or RTO question?

| Question asks about... | Variable | Drives... |
|---|---|---|
| "How much data can we lose?" / backup frequency / replication type | RPO | Backup schedule, replication mode (sync/async), log shipping interval |
| "How fast must we recover?" / architecture / clustering | RTO | DR technique, budget, automation (HA vs manual failover vs cold restore) |

### Filter 3: Is it a boundary question?

| Pair | Distinguishing property |
|---|---|
| RPO vs RTO | Data loss tolerance vs downtime tolerance |
| Differential vs Incremental | Since last FULL vs since last backup OF ANY TYPE |
| Synchronous vs Asynchronous | Primary waits for ACK (safe, slow) vs commits immediately (fast, risky) |
| Replication vs Mirroring | Both copy data continuously; mirroring is specifically DB-level real-time copy. Exam often treats them similarly — focus on sync vs async distinction. |
| Replication vs Log shipping | Continuous stream vs periodic file transfer |
| HA clustering vs Replication | Clustering = multiple nodes, auto-failover, shared workload. Replication = standby server receiving copies |
| Active-passive vs Active-active | Standby is idle until needed vs all nodes serve traffic |
| Failover vs Failback | Switch TO standby vs switch BACK to restored primary |
| On-site vs Off-site storage | Fast restore vs disaster survival |
| Purge vs Archive | Delete after period vs move to long-term cold storage |
| Full vs Differential vs Incremental | Restore: 1 file vs 2 files vs N files in sequence |
| Physical backup vs Database dump | Raw file copy (fast, platform-specific) vs SQL export (slow, portable) |
| Tabletop vs Simulation vs Full test | Paper walkthrough vs test environment vs production failover |
| Offline vs Immutable backups | Air-gapped (no network) vs WORM (can't be modified even by admin) |

### Filter 4: Is it a sequence question?

| Process | Correct order | Exam trap |
|---|---|---|
| Failback | Verify primary → Resync data → Validate integrity → Switch traffic → Monitor → Document | Switching traffic before resyncing or validating |
| Backup validation | Schedule → Complete → Hash at creation → Test restore periodically → Validate hash before restore | Never testing restores. Skipping hash validation. |
| Restore from incremental | Full + Inc1 + Inc2 + Inc3... in order | Applying incrementals out of order or skipping one |
| Restore from differential | Full + latest differential only | Using anything other than the latest differential |

### Filter 5: Is it a "what's wrong?" diagnostic question?

| Scenario | Problem | Fix |
|---|---|---|
| "RPO is 15 min but backups run daily" | Backup frequency doesn't meet RPO | Increase frequency (log backups or replication every 15 min) |
| "Backup completed but file is 0 bytes" | Backup failed silently | Investigate job; validate file size in monitoring alerts (D3) |
| "Backup restored but hash doesn't match" | Corruption or tampering | Do NOT restore from this file; use a different backup copy |
| "DR plan worked but failback took 3 days" | Failback process not tested/optimized | Practice failback; document and improve the process |
| "Ransomware encrypted live DB and backups" | Backups were online/accessible | Implement offline or immutable backups |
| "Archived backup can't be read" | Encryption key was not retained | Retain keys for same duration as backup; document key storage |
| "RTO is 5 min but using log shipping" | Log shipping RTO is minutes-hours | Need HA clustering or sync replication for 5-min RTO |

---

## 10. Domain 5 Ripple Effects

| If this fails... | Cascade through the system... |
|---|---|
| **No RPO defined** | → No backup frequency target → Backups run "whenever" → Data loss during disaster is unpredictable → Business discovers it lost 3 days of data → Unacceptable, but no one defined "unacceptable" in advance |
| **Backups never tested** | → Backup files exist on disk → Disaster occurs → DBA attempts restore → File is corrupt / missing pages / encryption key unavailable → Recovery FAILS → Data loss that was entirely preventable |
| **Only on-site backups** | → Fire/flood destroys data center → Primary DB: gone → Backup files: also gone (same building) → Recovery: IMPOSSIBLE → Business continuity plan fails at the most fundamental level |
| **Synchronous replication chosen without understanding latency** | → Every write waits for standby ACK → Application response time doubles → Users complain about slowness → DBA switches to async under pressure → RPO increases from ~zero to seconds-minutes → Business wasn't informed of the trade-off |
| **Log shipping interval too long for RPO** | → RPO is 15 minutes → Log shipping runs hourly → Disaster at minute 59 → 59 minutes of data lost → RPO violated by 4x → Recovery meets timing but not data requirements |
| **Failback without resyncing data** | → Primary restored, traffic switched back → But primary has stale data (pre-disaster) → DR site had newer data → Two versions of truth → Data integrity compromised → Hours of manual reconciliation |
| **Encryption keys not retained with archived backups** | → 7-year-old backup needed for legal discovery → Backup file exists in cold storage → Encryption key was purged after 1 year → Backup is permanently unreadable → Legal obligation unmet → Consequences |
| **DR plan never updated after events** | → Same procedural gaps persist → Next disaster triggers same failures → Recovery takes longer each time because team forgets and re-learns → Organization never improves |
| **No immutable/offline backups** | → Ransomware compromises admin credentials → Ransomware encrypts live DB → Then encrypts all network-accessible backups → ALL copies are encrypted → Only option: pay ransom (no guarantee) or accept total data loss |
| **Tabletop exercise reveals gaps but nobody remediates** | → Known gaps documented → Filed and forgotten → Actual disaster triggers exact failure predicted by tabletop → Team realizes the gap was identified months ago → Preventable loss of data and credibility |
