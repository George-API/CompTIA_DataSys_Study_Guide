# Domain 1 — Database Fundamentals: The Design System

Domain 1 is not four separate objectives. It is one **design pipeline** — a sequence of decisions where each choice constrains every choice that follows. The first decision (relational or NoSQL) determines your query language, scaling model, consistency guarantees, schema strategy, programming objects, scripting tools, and ORM behavior. Nothing in Domain 1 is independent.

This guide maps the internal system of Domain 1 so you can trace any concept back to the decision that created it.

---

## 1. The Design Pipeline

Every database begins as a business need and ends as a running, documented system. Domain 1 covers the first half of this pipeline — everything before deployment.

```
  REQUIREMENTS                ARCHITECTURE               DATA MODEL
  What do we need?            What constraints exist?     How should data be structured?
  ┌──────────────┐            ┌──────────────┐            ┌──────────────┐
  │ Users        │            │ Cloud vs     │            │ Relational   │
  │ Storage      │───────────►│ on-prem      │───────────►│   or         │
  │ Objectives   │            │ IaaS/PaaS/   │            │ NoSQL?       │
  │ (OLTP/OLAP)  │            │ SaaS         │            │ Which type?  │
  └──────────────┘            └──────────────┘            └──────────────┘
                                                                │
       ┌────────────────────────────────────────────────────────┘
       │
       ▼
  SCHEMA                     SQL & OBJECTS               ABSTRACTION
  How is it organized?       How do we interact with it? How does app code connect?
  ┌──────────────┐            ┌──────────────┐            ┌──────────────┐
  │ Logical      │            │ DDL / DML /  │            │ ORM          │
  │   → Physical │───────────►│ DCL / TCL    │───────────►│ Scripting    │
  │   → View     │            │ Triggers,    │            │ Server/client│
  │              │            │ views, procs │            │ side         │
  └──────────────┘            └──────────────┘            └──────────────┘
                                                                │
                                                                ▼
                                                          DOCUMENTATION
                                                          ┌──────────────┐
                                                          │ Data dict    │
                                                          │ ERD          │
                                                          │ Sys req docs │
                                                          └──────────────┘
```

**The key insight:** Exam questions about Domain 1 always sit somewhere on this pipeline. Identify *where* on the pipeline the question lives, and the answer space narrows to 2-3 possibilities.

---

## 2. Governing Principles of Database Design

### Principle 1: The First Decision Determines Everything

Choosing relational vs NoSQL is not a preference — it's a **constraint cascade**:

```
                          RELATIONAL                          NoSQL
                          ┌─────────────────────┐             ┌─────────────────────┐
  Decision made ─────────►│ Schema-on-WRITE      │             │ Schema-on-READ       │
                          │                     │             │                     │
  Which means... ────────►│ Schema defined       │             │ Structure defined    │
                          │ BEFORE data entry    │             │ at QUERY TIME        │
                          │                     │             │                     │
  Which forces... ───────►│ Fixed schema:        │             │ Flexible schema:     │
                          │ tables, columns,     │             │ documents, keys,     │
                          │ data types, FKs      │             │ column families      │
                          │                     │             │                     │
  Which enables... ──────►│ ACID transactions    │             │ BASE consistency     │
                          │ Referential integrity│             │ Eventual consistency │
                          │ Complex JOINs        │             │ Denormalized data    │
                          │                     │             │                     │
  Which requires... ─────►│ SQL (standardized)   │             │ Vendor-specific APIs │
                          │ DDL + DML + DCL + TCL│             │ Cypher, Gremlin,     │
                          │                     │             │ MongoDB query lang   │
                          │                     │             │                     │
  Which scales via... ───►│ VERTICAL             │             │ HORIZONTAL           │
                          │ (bigger server)      │             │ (more nodes)         │
                          └─────────────────────┘             └─────────────────────┘
```

**Exam implication:** If a question mentions JOINs, foreign keys, ACID, or regulatory compliance → relational. If it mentions flexible schema, horizontal scaling, or massive unstructured volume → NoSQL.

### Principle 2: Every Abstraction Layer Hides Complexity — and Creates Risk

Domain 1 is a stack of abstractions. Each layer simplifies the one below it, but every abstraction can leak:

```
    ORM (Hibernate, Entity Framework, Ebean)
         ↑ abstracts away
    SQL (DDL, DML, DCL, TCL)
         ↑ abstracts away
    Schema (logical → physical → view)
         ↑ abstracts away
    Storage (SSD, HDD, SAN, NAS, DAS)
         ↑ abstracts away
    Hardware (CPU, RAM, IOPS)
```

When the ORM generates an N+1 query, the abstraction has leaked — the problem originates at the ORM layer but manifests as CPU/memory pressure at the hardware layer. The DBA's job is to **trace upward through the stack** to find the actual source.

### Principle 3: SQL Is Set-Based, Not Row-Based

This single concept explains multiple exam answers:

| Situation | Set-based (correct) | Row-based (wrong) |
|---|---|---|
| Update 1000 rows | `UPDATE ... SET ... WHERE ...` (one operation) | Cursor loop processing one row at a time |
| Combine two result sets | `UNION` or `JOIN` | Application-side loop merging arrays |
| Remove all rows | `TRUNCATE` (DDL, whole set) or `DELETE` with no WHERE (DML, logged) | Loop with individual DELETEs |
| Find common records | `INTERSECT` | Nested loops comparing each row |

**The rule:** If the exam presents a cursor/loop solution alongside a set-based solution, the set-based solution is always the performance answer.

### Principle 4: Schema-on-Write vs Schema-on-Read Is the Fundamental Split

This isn't just a comparison row in a table — it's the philosophical divide that creates every other difference:

```
  SCHEMA-ON-WRITE (Relational)              SCHEMA-ON-READ (NoSQL)

  ┌─────────────────────────────┐           ┌─────────────────────────────┐
  │ "Define structure, then     │           │ "Store data, interpret      │
  │  insert data that fits"     │           │  structure when reading"    │
  └──────────────┬──────────────┘           └──────────────┬──────────────┘
                 │                                         │
  Validation happens at:                    Validation happens at:
  WRITE time (INSERT/UPDATE)                READ time (query/application)
                 │                                         │
  Rejects bad data immediately              Accepts anything; problems
                 │                          surface later at query time
                 │                                         │
  Schema changes are EXPENSIVE              Schema changes are CHEAP
  (ALTER TABLE, migration scripts)          (just store new shape)
                 │                                         │
  Guarantees: strong consistency            Guarantees: availability,
  referential integrity, ACID               partition tolerance (CAP)
  └─────────────────────────────┘           └─────────────────────────────┘
```

### Principle 5: Every Programming Object Solves a Specific Problem

Triggers, stored procedures, functions, and views are not interchangeable. Each exists because a specific problem cannot be solved by the others:

| Object | The problem it solves | Why nothing else can |
|---|---|---|
| **Trigger** | "Something must happen automatically every time data changes" | No other mechanism fires without being called |
| **Stored procedure** | "Complex logic must run server-side, called on demand" | Only object that accepts parameters and can contain control flow |
| **Function** | "A calculation must be reusable inside a SELECT" | Only object usable inline within queries |
| **View** | "Users need a simplified or restricted window into data" | Only object that presents a virtual table without storing data |

### Principle 6: Storage Architecture Determines Performance Ceiling

No amount of query optimization can overcome a fundamentally wrong storage choice. The hierarchy:

```
  PERFORMANCE (IOPS)

  HIGH ▲  ┌──────────────────────────────────────┐
       │  │ SAN (Storage Area Network)            │  Block-level, dedicated network
       │  │ High IOPS, high cost, enterprise      │  Best for databases
       │  ├──────────────────────────────────────┤
       │  │ SSD (Solid State Drive)               │  No moving parts, fast random I/O
       │  │ High IOPS, moderate cost              │  Preferred for active DB files
       │  ├──────────────────────────────────────┤
       │  │ NAS (Network-Attached Storage)        │  File-level, over network
       │  │ Moderate IOPS, good for file shares   │  Acceptable for backups, not live DB
       │  ├──────────────────────────────────────┤
       │  │ DAS (Direct-Attached Storage)         │  Physically attached to one server
       │  │ Moderate IOPS, low cost, no sharing   │  Single-server setups only
       │  ├──────────────────────────────────────┤
       │  │ HDD (Hard Disk Drive)                 │  Spinning platters, slow random I/O
       │  │ Low IOPS, lowest cost                 │  Archival, cold storage
  LOW  ▼  └──────────────────────────────────────┘

  COST follows the same order: SAN > SSD > NAS > DAS > HDD
```

**Exam implication:** "The database requires high IOPS for transactional workloads" → SSD or SAN. "Budget-friendly archival storage" → HDD. "File-level shared storage for backups" → NAS.

---

## 3. The Database Selection Decision Tree

When the exam describes a scenario and asks which database type to use, run through this tree:

```
  START: What does the workload need?
    │
    ├── Strict referential integrity? Complex JOINs? Regulatory compliance?
    │   ACID required?
    │     │
    │     YES ──────────────────────────────────► RELATIONAL (SQL)
    │                                              │
    │                                              ├── Transactional (OLTP)?
    │                                              │   → Normalized schema
    │                                              │   → Optimized for writes
    │                                              │
    │                                              └── Analytical (OLAP)?
    │                                                  → Denormalized / star schema
    │                                                  → Optimized for reads
    │
    └── NO strict schema needed? High volume? Flexible structure?
        │
        ├── Data is JSON-like documents with varying fields?
        │   → DOCUMENT DB (MongoDB, Cosmos)
        │
        ├── Simple lookups by unique key? Caching? Sessions?
        │   → KEY-VALUE STORE (DynamoDB, Redis)
        │
        ├── High write throughput? Time-series? Wide rows?
        │   → COLUMN-ORIENTED (Cassandra, HBase)
        │
        └── Traversing relationships IS the query? (friends-of-friends,
            fraud paths, recommendation chains)
            → GRAPH DB (Neo4j, Cosmos Gremlin API)
```

### Visual: NoSQL Types — What Makes Each One Different

The four NoSQL types are not variations of the same thing. They solve fundamentally different data problems:

```
  DOCUMENT                          KEY-VALUE
  ┌──────────────────────┐          ┌──────────────────────┐
  │ { "name": "Lee",     │          │ key_001 → [blob]     │
  │   "orders": [        │          │ key_002 → [blob]     │
  │     { "item": "A" }, │          │ key_003 → [blob]     │
  │     { "item": "B" }  │          │                      │
  │   ]                  │          │ Simplest model.      │
  │ }                    │          │ No structure inside   │
  │                      │          │ the value. Just GET   │
  │ Rich queries INTO    │          │ and PUT by key.       │
  │ the document.        │          │                      │
  │ Flexible per-doc     │          │ Fastest possible     │
  │ schema.              │          │ lookups.             │
  └──────────────────────┘          └──────────────────────┘
  MongoDB, Cosmos                   DynamoDB, Redis

  COLUMN-ORIENTED                   GRAPH
  ┌──────────────────────┐          ┌──────────────────────┐
  │  Row   Col1  Col2    │          │   (Alice)            │
  │  ────  ────  ────    │          │     │                │
  │  R1    A     X       │          │   KNOWS              │
  │  R2    B     Y       │          │     │                │
  │  R3    C     Z       │          │   (Bob)──KNOWS──(Eve)│
  │                      │          │                      │
  │ Stored BY COLUMN,    │          │ Nodes + edges +      │
  │ not by row.          │          │ properties.          │
  │ Compress well.       │          │                      │
  │ Write-optimized      │          │ Traversal IS the     │
  │ (append to column    │          │ query. "Find all     │
  │ family).             │          │ friends-of-friends"  │
  └──────────────────────┘          └──────────────────────┘
  Cassandra, HBase                  Neo4j (Cypher), Cosmos (Gremlin)
```

---

## 4. SQL as One Coherent System

The four sublanguages are not independent categories to memorize. They are four subsystems that work together within a single operation lifecycle:

```
  ┌─────────────────────────────────────────────────────────────────────┐
  │                    THE SQL OPERATION LIFECYCLE                      │
  │                                                                     │
  │  1. DDL creates the structure                                      │
  │     CREATE TABLE → ALTER TABLE → (eventually DROP / TRUNCATE)      │
  │              │                                                      │
  │              ▼                                                      │
  │  2. DCL controls who can touch it                                  │
  │     GRANT SELECT ON table TO analyst_role                          │
  │     REVOKE INSERT ON table FROM intern_role                        │
  │              │                                                      │
  │              ▼                                                      │
  │  3. DML operates on the data                                       │
  │     INSERT / SELECT / UPDATE / DELETE                              │
  │              │                                                      │
  │              ▼                                                      │
  │  4. TCL wraps DML in safety                                        │
  │     BEGIN TRANSACTION                                              │
  │       UPDATE accounts SET balance = balance - 100 WHERE id = 1;   │
  │       UPDATE accounts SET balance = balance + 100 WHERE id = 2;   │
  │     COMMIT;  ── or ──  ROLLBACK;                                   │
  │                                                                     │
  │  TCL enforces ACID:                                                │
  │    Atomicity  → both UPDATEs succeed or both roll back             │
  │    Consistency → constraints checked after each statement          │
  │    Isolation  → other transactions don't see partial changes       │
  │    Durability → once COMMIT returns, data survives a crash         │
  └─────────────────────────────────────────────────────────────────────┘
```

### The ACID Properties as a System

ACID is not four independent properties. They form a dependency chain:

```
  Durability
       ↑ requires
  Atomicity (if a crash happens mid-transaction, can we roll back cleanly?)
       ↑ requires
  Isolation (if two transactions run concurrently, can atomicity be violated?)
       ↑ requires
  Consistency (if isolation fails, can constraints be violated?)
```

Remove any one property and the ones above it become unreliable. This is why relational databases enforce all four together — they are a single guarantee, not a checklist.

### Visual: TRUNCATE vs DELETE — The Boundary the Exam Loves

```
  TRUNCATE                                    DELETE
  ┌──────────────────────────────┐            ┌──────────────────────────────┐
  │  DDL (Data DEFINITION)       │            │  DML (Data MANIPULATION)      │
  │                              │            │                              │
  │  TRUNCATE TABLE employees;   │            │  DELETE FROM employees        │
  │                              │            │  WHERE dept = 'Sales';        │
  │  • Removes ALL rows          │            │                              │
  │  • Cannot use WHERE          │            │  • Can remove specific rows  │
  │  • Minimal logging           │            │  • CAN use WHERE             │
  │  • Resets identity/auto-inc  │            │  • Fully logged (rollback OK)│
  │  • Fastest for "clear all"   │            │  • Fires DELETE triggers     │
  │  • Does NOT fire triggers    │            │  • Slower for large tables   │
  │                              │            │                              │
  │  Think: "RESET the table"    │            │  Think: "REMOVE specific     │
  │  (structure stays, data gone)│            │   rows from the table"       │
  └──────────────────────────────┘            └──────────────────────────────┘

  THE EXAM TRAP:
  "Remove all data" could be either.
  Key word in the question:
    • "minimal logging" or "reset" → TRUNCATE
    • "audit trail" or "trigger must fire" or "WHERE" → DELETE
    • "Which sublanguage?" TRUNCATE = DDL, DELETE = DML
```

### Visual: Set Operations — How They Combine Results

```
  Query A returns:     Query B returns:
  ┌────┐               ┌────┐
  │ 1  │               │ 2  │
  │ 2  │               │ 3  │
  │ 3  │               │ 4  │
  └────┘               └────┘

  A UNION B          A INTERSECT B       A EXCEPT B
  ┌────┐             ┌────┐              ┌────┐
  │ 1  │             │ 2  │              │ 1  │
  │ 2  │             │ 3  │              └────┘
  │ 3  │             └────┘
  │ 4  │             Rows in BOTH        Rows in A
  └────┘                                 but NOT in B
  All rows,
  duplicates removed

  UNION ALL: same as UNION but KEEPS duplicates (1, 2, 3, 2, 3, 4)
```

---

## 5. The Abstraction Ladder

Domain 1 spans six levels of abstraction. Each level wraps the one below it. Understanding which level a question targets is half the answer.

```
  LEVEL 6: ORM                      Furthest from the database
  ┌──────────────────────────────────────────────────────────────┐
  │  Application code: customer.save()                          │
  │  Developer never writes SQL. ORM generates it.              │
  │  Risk: N+1 queries, over-fetching, implicit transactions    │
  │                                                              │
  │  Hibernate (Java/JPA) · Entity Framework (.NET) · Ebean     │
  └────────────────────────────┬─────────────────────────────────┘
                               │ generates
  LEVEL 5: SCRIPTING           ▼
  ┌──────────────────────────────────────────────────────────────┐
  │  Automation layer: schedule, execute, pipe results           │
  │                                                              │
  │  Server-side: cron + pg_dump, SQL Agent, scheduled ETL      │
  │  Client-side: ad-hoc queries, local exports, front-end      │
  │                                                              │
  │  PowerShell (Windows) · Python (cross-platform)             │
  │  Bash/cron (Linux) · CMD/Task Scheduler (Windows)           │
  └────────────────────────────┬─────────────────────────────────┘
                               │ executes
  LEVEL 4: PROGRAMMING OBJECTS ▼
  ┌──────────────────────────────────────────────────────────────┐
  │  Server-side SQL logic: triggers, stored procs, functions,  │
  │  views. Encapsulate business rules inside the database.     │
  └────────────────────────────┬─────────────────────────────────┘
                               │ composed of
  LEVEL 3: SQL                 ▼
  ┌──────────────────────────────────────────────────────────────┐
  │  DDL (structure) · DML (data) · DCL (access) · TCL (safety) │
  │  Set-based operations on tables. ANSI standard.             │
  └────────────────────────────┬─────────────────────────────────┘
                               │ operates on
  LEVEL 2: SCHEMA              ▼
  ┌──────────────────────────────────────────────────────────────┐
  │  Logical (entities, attributes, relationships)               │
  │  Physical (tables, indexes, partitions, storage engine)      │
  │  View (virtual tables, column subsets, security boundary)    │
  └────────────────────────────┬─────────────────────────────────┘
                               │ stored on
  LEVEL 1: STORAGE             ▼                  Closest to hardware
  ┌──────────────────────────────────────────────────────────────┐
  │  SSD · HDD · SAN · NAS · DAS                                │
  │  IOPS determines the performance ceiling.                   │
  └──────────────────────────────────────────────────────────────┘
```

**The debugging rule:** When something is slow or broken, trace **downward** through the ladder. The problem reported at one level often has its root cause one or two levels below:

| Symptom at level | Root cause often at level |
|---|---|
| ORM is slow (6) | Generated SQL is bad (3) or indexes are missing (2) |
| Script times out (5) | Query inside the script lacks a WHERE clause (3) |
| Stored proc is slow (4) | Execution plan shows table scan — missing index (2) |
| Query is slow (3) | Physical schema lacks proper indexing (2) or storage IOPS is saturated (1) |

---

## 6. Programming Objects as an Interconnected System

Triggers, stored procedures, functions, and views are not independent tools. They form a system with specific interaction rules:

```
                                 ┌──────────────────┐
                                 │  APPLICATION      │
                                 │  (or DBA console) │
                                 └────────┬─────────┘
                                          │
                          CALLS           │           SELECTS FROM
                   ┌──────────────────────┼──────────────────────┐
                   │                      │                      │
                   ▼                      │                      ▼
  ┌────────────────────────┐              │      ┌────────────────────────┐
  │   STORED PROCEDURE     │              │      │        VIEW            │
  │                        │              │      │                        │
  │  • Called BY NAME      │              │      │  • Virtual table       │
  │  • Accepts parameters  │              │      │  • Defined by SELECT   │
  │  • Can contain DDL,    │              │      │  • No data stored      │
  │    DML, control flow   │              │      │  • Always current      │
  │  • Reduces round-trips │              │      │  • Hides complexity    │
  │  • Security via        │              │      │  • Restricts columns   │
  │    EXECUTE permission  │              │      │    (confidentiality)   │
  └────────┬───────────────┘              │      └────────────────────────┘
           │                              │
           │ executes DML (INSERT/UPDATE/DELETE)
           │                              │
           ▼                              ▼
  ┌────────────────────────────────────────────────────────────┐
  │                         TABLE                              │
  │                                                            │
  │  Data changes trigger...                                   │
  └────────────────────────────┬───────────────────────────────┘
                               │ automatically fires
                               ▼
  ┌────────────────────────┐              ┌────────────────────────┐
  │       TRIGGER          │              │       FUNCTION         │
  │                        │              │                        │
  │  • Fires on INSERT,    │              │  • Returns a value     │
  │    UPDATE, or DELETE   │              │  • Usable INSIDE       │
  │  • NEVER called by     │              │    SELECT statements   │
  │    name                │              │  • Scalar: one value   │
  │  • Audit logging       │              │  • Table: result set   │
  │  • Enforce rules the   │              │  • Reusable calc       │
  │    schema can't express│              │                        │
  └────────────────────────┘              └────────────────────────┘
                                              ▲
                                              │ called within
                                     SQL queries, views,
                                     stored procedures
```

### The Decision Rule for Programming Objects

```
  Question describes...                              Answer is...

  "automatically log every UPDATE" ──────────────────► TRIGGER
  "fires whenever a row changes" ────────────────────► TRIGGER

  "application calls a reusable routine" ────────────► STORED PROCEDURE
  "encapsulate business logic server-side" ──────────► STORED PROCEDURE
  "reduce network round-trips" ──────────────────────► STORED PROCEDURE

  "calculated column inside a SELECT" ───────────────► FUNCTION
  "reusable formula in queries" ─────────────────────► FUNCTION

  "hide complexity from end users" ──────────────────► VIEW
  "restrict which columns a role can see" ───────────► VIEW
  "stable interface while schema changes underneath" ► VIEW
```

---

## 7. The ORM Gap — Where Abstraction Breaks Down

The ORM sits at the top of the abstraction ladder. It lets developers write `customer.getOrders()` instead of SQL. The problem: the ORM makes SQL decisions the developer never sees, and those decisions can be catastrophic for the database.

### The Three ORM Anti-Patterns

```
  ANTI-PATTERN 1: N+1 QUERIES
  ─────────────────────────────────────────────────────────────
  Developer writes:  for customer in customers: print(customer.orders)

  ORM generates:     SELECT * FROM customers;                    ← 1 query
                     SELECT * FROM orders WHERE customer_id = 1; ← +1
                     SELECT * FROM orders WHERE customer_id = 2; ← +1
                     SELECT * FROM orders WHERE customer_id = 3; ← +1
                     ... (×N customers)                          ← = N+1 total

  Should be:         SELECT c.*, o.*
                     FROM customers c
                     JOIN orders o ON c.id = o.customer_id;      ← 1 query


  ANTI-PATTERN 2: OVER-FETCHING (SELECT *)
  ─────────────────────────────────────────────────────────────
  Developer writes:  customer = Customer.find(1)

  ORM generates:     SELECT * FROM customers WHERE id = 1;
                     ↑ returns ALL 47 columns

  Developer uses:    customer.name, customer.email
                     ↑ only needed 2 columns

  Should be:         SELECT name, email FROM customers WHERE id = 1;


  ANTI-PATTERN 3: IMPLICIT TRANSACTIONS
  ─────────────────────────────────────────────────────────────
  ORM silently wraps operations in transactions.
  If the application pauses mid-operation (waiting for user input,
  calling an external API), the transaction stays OPEN.

  Open transaction → holds locks → other queries wait →
  lock contention → deadlock risk → throughput drops
```

### The ORM Impact Assessment Process (Exam-Critical Sequence)

This is a **strict order** — the exam tests "what should the DBA do FIRST?"

```
  Step 1                Step 2              Step 3              Step 4
  ┌──────────────┐      ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
  │ REVIEW       │      │ CONFIRM      │    │ DETERMINE    │    │ PROVIDE      │
  │ generated SQL│─────►│ validity of  │───►│ impact to    │───►│ solutions /  │
  │              │      │ code         │    │ DB server    │    │ alternate    │
  │ Enable query │      │              │    │              │    │ approach     │
  │ logging or   │      │ Correct      │    │ Execution    │    │              │
  │ profiling to │      │ syntax?      │    │ plans, CPU,  │    │ Query hints, │
  │ capture the  │      │ Proper index │    │ memory, lock │    │ eager load,  │
  │ actual SQL   │      │ use? Missing │    │ waits, I/O   │    │ stored procs,│
  │ hitting the  │      │ WHERE? Cart- │    │              │    │ raw SQL for  │
  │ server       │      │ esian joins? │    │              │    │ hot paths    │
  └──────────────┘      └──────────────┘    └──────────────┘    └──────────────┘
        ▲
        │
  ALWAYS the first step.
  "Application is slow after ORM update" → Step 1: review the generated SQL.
  NOT: add hardware, change schema, or guess.
```

---

## 8. Cardinality and Relationships — The Structural Foundation

Cardinality defines how entities relate. It constrains everything that follows: schema design, JOIN strategy, index placement, and foreign key behavior.

```
  ONE-TO-ONE (1:1)
  ┌──────────┐         ┌──────────┐
  │ Employee │────1:1──│ Badge    │     Each employee has exactly one badge.
  └──────────┘         └──────────┘     Badge FK → Employee PK (unique).
                                        Often merged into one table unless
                                        access control requires separation.

  ONE-TO-MANY (1:M)
  ┌──────────┐         ┌──────────┐
  │ Customer │────1:M──│ Order    │     One customer has many orders.
  └──────────┘         └──────────┘     Order table gets FK → Customer PK.
                                        Most common relationship type.
                                        JOIN direction: from many to one.

  MANY-TO-MANY (M:M)
  ┌──────────┐    ┌──────────────┐    ┌──────────┐
  │ Student  │──M:┤ Enrollment   ├:M──│ Course   │
  └──────────┘    │ (junction)   │    └──────────┘
                  └──────────────┘
  Requires a JUNCTION TABLE (also called bridge, associative, linking table).
  Junction table has two FKs: one to each parent.
  No direct M:M relationship in a relational schema — always decomposed.
```

**Exam implication:** If a question describes a relationship where "many of A relate to many of B," the answer involves a junction table. If the question says "each A has exactly one B," consider whether a separate table is even necessary.

---

## 9. Cloud Models — The Responsibility Spectrum

```
                   YOU MANAGE                        PROVIDER MANAGES
  ┌────────────────────────────────────┬────────────────────────────────────┐
  │                                    │                                    │
  │  IaaS (Infrastructure as a Service)│                                    │
  │  ┌──────────────────────────────┐  │  Provider gives you:              │
  │  │ YOU: OS, DBMS, schema, data, │  │  ┌──────────────────────────────┐│
  │  │ patching, backups, security  │  │  │ Hardware, network,           ││
  │  │                              │  │  │ virtualization               ││
  │  │ "Rent a VM, install your DB" │  │  └──────────────────────────────┘│
  │  └──────────────────────────────┘  │                                    │
  ├────────────────────────────────────┼────────────────────────────────────┤
  │                                    │                                    │
  │  PaaS (Platform as a Service)      │                                    │
  │  ┌──────────────────┐             │  ┌──────────────────────────────┐  │
  │  │ YOU: schema, data│             │  │ Provider: hardware, OS,     │  │
  │  │                  │             │  │ DBMS patching, backups      │  │
  │  │ "Managed DB"     │             │  │                              │  │
  │  │ Azure SQL, RDS   │             │  │ DBaaS is a PaaS offering    │  │
  │  └──────────────────┘             │  └──────────────────────────────┘  │
  ├────────────────────────────────────┼────────────────────────────────────┤
  │                                    │                                    │
  │  SaaS (Software as a Service)      │                                    │
  │  ┌──────────┐                     │  ┌──────────────────────────────┐  │
  │  │ YOU:     │                     │  │ Provider: EVERYTHING         │  │
  │  │ nothing  │                     │  │ Hardware, OS, DBMS, schema,  │  │
  │  │ (user)   │                     │  │ application, maintenance     │  │
  │  └──────────┘                     │  └──────────────────────────────┘  │
  └────────────────────────────────────┴────────────────────────────────────┘

  CONTROL ◄─────────────────────────────────────────────► CONVENIENCE
  IaaS          (most control, most work)
                     PaaS     (balanced)
                                    SaaS  (no control, no work)
```

**Exam decision rule:**
- "DBA needs full control over OS and DBMS configuration" → IaaS
- "DBA manages schema and data; provider handles patching" → PaaS
- "No database administration needed; users just access the application" → SaaS

---

## 10. Scripting as the Automation Layer

Scripting (1.2) is not a standalone topic. It's the mechanism that automates everything else in Domain 1 — and connects directly to Domain 3 (OPERATE) maintenance tasks.

```
  WHAT SCRIPTING AUTOMATES IN DOMAIN 1:

  ┌─────────────────────────────────────────────────────┐
  │ Schema creation    │  DDL scripts: CREATE, ALTER     │
  │ Data loading       │  DML scripts: INSERT, bulk load │
  │ Index management   │  DDL: CREATE INDEX, rebuild     │
  │ Backup scheduling  │  pg_dump, mysqldump, sqlcmd     │
  │ ETL pipelines      │  Extract → Transform → Load     │
  │ Health checks      │  Query system views, alert      │
  └─────────────────────────────────────────────────────┘

  WHERE IT RUNS:

  ┌─────────────────────────────┐    ┌─────────────────────────────┐
  │     SERVER-SIDE             │    │     CLIENT-SIDE             │
  │                             │    │                             │
  │  Runs on DB or app server   │    │  Runs on user's workstation │
  │                             │    │                             │
  │  + Closer to data (fast)    │    │  + Quick ad-hoc queries     │
  │  + Credentials stay on      │    │  - Data travels over        │
  │    server (secure)          │    │    network (slow, risky)    │
  │  + Easy to schedule         │    │  - Credential exposure      │
  │    (cron, Task Scheduler,   │    │    risk                     │
  │    SQL Agent)               │    │  - Requires user action     │
  │                             │    │    (or separate scheduler)  │
  │  Preferred for:             │    │  Preferred for:             │
  │  maintenance, backups,      │    │  ad-hoc analysis, local    │
  │  ETL, scheduled tasks       │    │  exports, development      │
  └─────────────────────────────┘    └─────────────────────────────┘

  TOOLS BY PLATFORM:

  ┌──────────────────────────────────────────────────────────────┐
  │  LINUX                        │  WINDOWS                     │
  │                               │                              │
  │  Bash                         │  PowerShell                  │
  │  cron (scheduling)            │  Task Scheduler              │
  │  psql (PostgreSQL CLI)        │  sqlcmd (SQL Server CLI)     │
  │  mysql (MySQL CLI)            │  Invoke-Sqlcmd (PowerShell)  │
  │  mongosh (MongoDB CLI)        │  SQL Server Agent            │
  │                               │                              │
  │  "Schedule nightly backup     │  "Schedule nightly backup    │
  │   on Linux?" → cron           │   on Windows?" →             │
  │                               │   Task Scheduler or          │
  │                               │   SQL Server Agent           │
  └──────────────────────────────────────────────────────────────┘
```

---

## 11. Domain 1 Decision Engine

When reading a Domain 1 exam question, run it through these filters:

### Filter 1: Where on the design pipeline?

| Question is about... | Pipeline stage | Sub-objectives |
|---|---|---|
| Users, storage needs, OLTP vs OLAP, gap analysis | Requirements | 1.4 |
| Cloud models, on-prem vs cloud, system specs | Architecture | 1.4 |
| Relational vs NoSQL, which NoSQL type | Data model | 1.0 |
| Logical, physical, view schema; ERD; data dictionary | Schema & docs | 1.4 |
| DDL, DML, DCL, TCL, ACID, JOINs, set operations | SQL | 1.1 |
| Triggers, stored procs, functions, views | Programming objects | 1.1 |
| PowerShell, Python, Bash, cron, server/client side | Scripting | 1.2 |
| ORM, N+1, over-fetching, impact assessment | Abstraction | 1.3 |

### Filter 2: Is it a "which tool/type" question?

| Scenario keywords | Answer |
|---|---|
| Flexible schema, JSON documents, web/mobile | MongoDB (Document DB) |
| Fast lookups by key, caching, sessions | DynamoDB / Redis (Key-Value) |
| High write throughput, time-series, analytics at scale | Cassandra (Column-Oriented) |
| Social network, fraud detection, relationship traversal | Neo4j (Graph DB) |
| Multi-model, Azure, supports multiple APIs | Cosmos DB |
| Schedule job on Linux | cron |
| Schedule job on Windows | Task Scheduler / SQL Server Agent |
| Automate SQL Server on Windows | PowerShell (Invoke-Sqlcmd) |
| Cross-platform scripting with DB libraries | Python |

### Filter 3: Is it a boundary question?

| Pair | The distinguishing property |
|---|---|
| TRUNCATE vs DELETE | DDL vs DML; no WHERE vs WHERE; no triggers vs fires triggers |
| Trigger vs Stored procedure | Automatic vs called by name |
| View vs Materialized view | No data stored (always current) vs physically stored (stale until refreshed) |
| Logical vs Physical schema | Platform-independent vs platform-specific |
| Schema-on-write vs Schema-on-read | Relational vs NoSQL |
| Server-side vs Client-side scripting | Credentials stay on server vs travel to client |
| UNION vs UNION ALL | Removes duplicates vs keeps duplicates |
| INTERSECT vs EXCEPT | Common to both vs in first but not second |
| Cursor vs Set-based | Row-by-row (slow) vs operates on whole set (fast) |
| OLTP vs OLAP | Transactional, normalized, write-optimized vs analytical, denormalized, read-optimized |
| Vertical vs Horizontal scaling | Bigger server (relational) vs more nodes (NoSQL) |

### Filter 4: Is it a sequence question?

| Process | Correct order | First step |
|---|---|---|
| ORM impact assessment | Review SQL → Confirm validity → Determine impact → Provide solutions | Review the generated SQL |
| Requirements gathering | Users → Storage → Objectives | Identify use cases |
| Schema design | Logical → Physical → View | Start with entities and relationships (logical) |

---

## 12. Domain 1 Ripple Effects

| If this design decision is wrong... | These downstream effects occur... |
|---|---|
| **Chose relational for unstructured, high-volume data** | → Schema constantly needs ALTER → developers fight constraints → horizontal scaling impossible → performance degrades as volume grows → eventually requires migration |
| **Chose NoSQL for data requiring strict referential integrity** | → Application code must enforce relationships → inconsistency bugs → audit failures → regulatory compliance risk |
| **Logical schema has no ERD** | → Physical schema built on assumptions → missing relationships → JOINs produce wrong results → data dictionary is incomplete → new team members misunderstand the data model |
| **No data dictionary maintained** | → Schema changes made without shared reference → column meanings drift → queries built on wrong assumptions → maintenance becomes guesswork |
| **Views not used to restrict columns** | → All users see all columns including sensitive data → confidentiality violation → fails PCI DSS / HIPAA audit |
| **Chose HDD when workload needs high IOPS** | → Every query hits the storage ceiling → indexing helps but can't overcome I/O bottleneck → users experience latency → no software fix exists (hardware problem) |
| **ORM anti-patterns ignored** | → N+1 queries explode under load → connection pool exhausted → CPU spike → other applications on same server affected → monitoring alerts cascade |
| **Cursor used instead of set-based SQL** | → Processing time scales linearly with row count → acceptable at 100 rows, catastrophic at 100,000 → locks held longer → deadlock risk increases |
