# 1.0 Database Fundamentals — 24%

---

## Compare and Contrast Database Structure Types

### Relational vs. Nonrelational Databases

| Aspect | Relational (SQL) | Nonrelational (NoSQL) |
|--------|------------------|-----------------------|
| **Data model** | Tables with rows and columns; fixed schema | Flexible schemas — documents, key-value pairs, graphs, wide columns |
| **Schema** | Schema-on-write (defined before data entry) | Schema-on-read (structure interpreted at query time) |
| **Relationships** | Enforced via foreign keys, JOINs | Typically denormalized; relationships handled in application code or specialized engines (graph DBs) |
| **Scaling** | Vertical (scale up — bigger server) | Horizontal (scale out — add nodes) |
| **ACID** | Full ACID by default | Varies — many favor eventual consistency (BASE model) |
| **Query language** | SQL (standardized) | Vendor-specific APIs or query languages |
| **Best for** | Structured, transactional workloads; strict integrity | High-velocity, high-volume, or semi-/unstructured data; rapid iteration |

**Exam tip:** If a question describes strict referential integrity, complex JOINs, or regulatory compliance, lean toward relational. If it mentions rapid schema changes, massive horizontal scaling, or unstructured data, lean NoSQL.

### Linear vs. Non-linear Format

| Format | Description | Example |
|--------|-------------|---------|
| **Linear** | Data stored/accessed sequentially — row after row, record after record | Flat files, CSV, traditional row-store RDBMS |
| **Non-linear** | Data stored in interconnected or hierarchical structures; traversal is not sequential | Graph databases, document stores, tree structures |

### NoSQL Types

| Type | How It Stores Data | Strengths | Common Tool |
|------|--------------------|-----------|-------------|
| **Document** | JSON/BSON documents; each document can have a different structure | Flexible schema, natural for web/mobile apps | **MongoDB**, **Cosmos** (document API) |
| **Key-value** | Simple pairs — a unique key maps to a blob/value | Extremely fast lookups, caching, session stores | **Amazon DynamoDB**, Redis |
| **Column-oriented** | Data organized by columns (column families) instead of rows | High write throughput, time-series, analytics at scale | **Cassandra**, HBase |
| **Graph** | Nodes + edges + properties; optimized for relationship traversal | Social networks, fraud detection, recommendation engines | **Neo4j**, **Cosmos** (Gremlin API) |

### Tools to Know

| Tool | Type | Key Fact |
|------|------|----------|
| **Cassandra** | Column-oriented | Apache project; masterless ring architecture; linear scalability |
| **MongoDB** | Document | BSON storage; rich query language; aggregation pipeline |
| **Neo4j** | Graph | Cypher query language; ACID-compliant graph DB |
| **Amazon DynamoDB** | Key-value (with document features) | Fully managed AWS service; single-digit ms latency at any scale |
| **Cosmos** | Multi-model | Azure service; supports document, key-value, graph, and column-family APIs |

**What they might ask:** "Which NoSQL type is best for modeling social-network relationships?" → Graph (Neo4j). "Which is best for high write throughput with time-series data?" → Column-oriented (Cassandra).

---

## 1.1 Given a Scenario, Develop, Modify, and Run SQL Code

This is a scenario-based objective — expect questions that present a situation and ask you to choose the correct SQL statement, identify an error, or pick the right approach.

### SQL Sublanguages

| Sublanguage | Purpose | Key Statements |
|-------------|---------|----------------|
| **DDL** (Data Definition Language) | Define/modify schema objects | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` |
| **DML** (Data Manipulation Language) | Manipulate data within tables | `SELECT`, `INSERT`, `UPDATE`, `DELETE` |
| **TCL** (Transaction Control Language) | Control transactions | `BEGIN TRANSACTION`, `COMMIT`, `ROLLBACK`, `SAVEPOINT` |

**Decision steps for scenario questions:**

1. **Are you changing structure or data?** Structure → DDL. Data → DML.
2. **Do you need to wrap multiple operations atomically?** Use TCL (`BEGIN … COMMIT/ROLLBACK`).
3. **Does the question mention removing all rows but keeping the table?** → `TRUNCATE` (DDL, no row-level logging) vs. `DELETE` (DML, logged, can use `WHERE`).

### Set-Based Logic

SQL operates on **sets of rows**, not row-by-row iteration. Key concepts:

- **UNION / UNION ALL** — combine result sets (UNION removes duplicates; UNION ALL keeps them)
- **INTERSECT** — rows common to both queries
- **EXCEPT** — rows in the first query but not the second
- **JOINs** — combine columns from related tables (INNER, LEFT, RIGHT, FULL OUTER, CROSS)

**Exam tip:** If a question contrasts a cursor-based loop with a single `UPDATE … SET … WHERE`, the set-based approach is almost always the correct answer for performance.

### ACID Principles

| Property | Meaning | Why It Matters |
|----------|---------|----------------|
| **Atomicity** | All operations in a transaction succeed or all roll back | No partial updates |
| **Consistency** | Transaction moves the DB from one valid state to another | Constraints, triggers, and rules are enforced |
| **Isolation** | Concurrent transactions don't interfere | Prevents dirty reads, phantom reads |
| **Durability** | Once committed, data survives crashes | Write-ahead logs, checkpoints |

### ANSI SQL

CompTIA tests vendor-neutral, **ANSI-standard SQL**. Key points:

- Standard syntax for `SELECT`, `INSERT`, `UPDATE`, `DELETE`, JOINs, subqueries
- Vendor extensions (T-SQL, PL/SQL) may appear in distractors — favor ANSI-compliant answers unless the scenario specifies a vendor

### Programming with SQL

| Object | What It Does | When to Use |
|--------|--------------|-------------|
| **Triggers** | Automatically fire SQL in response to `INSERT`, `UPDATE`, or `DELETE` events | Audit logging, enforcing complex business rules, cascading changes |
| **Stored procedures** | Precompiled SQL routines that accept parameters and can return results | Encapsulate business logic server-side, reduce network round-trips, enforce security via `EXECUTE` permissions |
| **Functions** | Return a single value (scalar) or a table; usable inside queries | Calculations, transformations, reusable logic within `SELECT` |
| **Views** | Virtual tables defined by a `SELECT` statement; no data stored separately (unless materialized) | Simplify complex queries, restrict column/row access, present a stable interface while underlying schema changes |

**Given a scenario checklist:**

- Need to enforce a rule every time a row changes? → **Trigger**
- Need a reusable block of logic called from an application? → **Stored procedure**
- Need a calculated column inside a `SELECT`? → **Function**
- Need to hide complexity or limit visible columns for a user role? → **View**

**Exam tip:** A question that says "the DBA wants to automatically log every UPDATE to an audit table" is pointing at a trigger, not a stored procedure.

---

## 1.2 Compare and Contrast Scripting Methods and Environments

### Script Purpose and Runtime Location

| Location | Description | Typical Use |
|----------|-------------|-------------|
| **Server-side** | Script runs on the database or application server | Automated maintenance (backups, index rebuilds), ETL jobs, scheduled tasks |
| **Client-side** | Script runs on the user's workstation or in the browser | Ad-hoc queries, local data exports, front-end validation |

**Trade-offs:**

| Factor | Server-side | Client-side |
|--------|-------------|-------------|
| Performance | Leverages server resources; closer to data | Limited by client hardware; data must travel over the network |
| Security | Credentials stay on server; tighter control | Risk of exposing connection strings or credentials |
| Scheduling | Easily automated via OS scheduler or DB agent | Requires user initiation or separate orchestration |

### Languages

| Language | Strengths for DB Work | Platform |
|----------|----------------------|----------|
| **PowerShell** | Native Windows automation; rich cmdlets for SQL Server (`Invoke-Sqlcmd`), Active Directory; object pipeline | Windows (cross-platform via PowerShell Core, but exam focus is Windows) |
| **Python** | Cross-platform; libraries for every major DBMS (`pyodbc`, `psycopg2`, `pymongo`); strong for ETL and data analysis | Windows, Linux, macOS |

### Command-Line Scripting

| OS | Shell / Tools | DB-Relevant Examples |
|----|---------------|---------------------|
| **Linux** | Bash, `cron`, `mysql`, `psql`, `mongosh` | `crontab -e` to schedule a nightly `pg_dump`; pipe output with `|`, redirect with `>` |
| **Windows** | CMD, PowerShell, `sqlcmd`, Task Scheduler | `sqlcmd -S server -d db -Q "SELECT …"`; PowerShell `Invoke-Sqlcmd` for scripted admin |

**What they might ask:** "A DBA needs to automate a weekly backup on a Linux server — which tool schedules the job?" → `cron`. On Windows? → Task Scheduler (or SQL Server Agent).

---

## 1.3 Explain the Impact of Programming on Database Operations

### Object-Relational Mapping (ORM)

An ORM maps application objects (classes) to database tables, letting developers interact with data using their programming language instead of raw SQL.

| ORM | Language / Framework | Key Fact |
|-----|---------------------|----------|
| **Hibernate** | Java / JPA | Mature, widely used; generates complex SQL including lazy/eager loading |
| **Entity Framework** | .NET (C#) | Microsoft ecosystem; Code-First and Database-First workflows |
| **Ebean** | Java | Lightweight alternative to Hibernate; simpler configuration |

### Why ORMs Matter to a DBA

ORMs can generate **inefficient SQL** without the developer (or DBA) realizing it:

- **N+1 query problem** — ORM issues one query per related object instead of a single JOIN
- **Over-fetching** — `SELECT *` when only two columns are needed
- **Implicit transactions** — ORM may hold transactions open longer than expected, increasing lock contention

### Process to Gauge Impact (Exam-Critical Sequence)

1. **Review SQL code generated by the ORM** — enable query logging or profiling to capture the actual SQL hitting the server.
2. **Confirm validity of code** — verify correct syntax, proper use of indexes, no Cartesian products or missing `WHERE` clauses.
3. **Determine impact to the database server** — assess execution plans, CPU/memory usage, lock waits, and I/O.
4. **Provide solutions / alternate approach, as needed** — suggest query hints, eager loading, stored procedures, or raw SQL for hot paths.

**Exam tip:** If the question states "application performance degrades after an ORM update," the first step is always to review the generated SQL — not to add hardware or change the schema.

---

## 1.4 Compare and Contrast Aspects of Database Planning and Design

### Requirements Gathering

| Factor | What to Determine | Why |
|--------|-------------------|-----|
| **Number of users** | Concurrent connections, peak load | Drives licensing, connection pooling, and hardware sizing |
| **Storage capacity** | Size (GB/TB), speed (IOPS, latency), type (SSD vs. HDD, SAN vs. NAS vs. DAS) | Determines storage architecture and cost |
| **Database objectives** | Use cases and purposes (OLTP, OLAP, mixed workload, data warehouse) | Shapes schema design, indexing strategy, and platform choice |

### Database Architecture Factors

| Factor | Details |
|--------|---------|
| **Inventory of needed assets** | Servers, storage, networking gear, licenses; perform a **gap analysis** to identify what's missing vs. available |
| **Cloud-based vs. on-premises** | Cloud: elastic scaling, managed services, OpEx model. On-prem: full control, CapEx model, compliance constraints |
| **Cloud-hosted environment types** | **PaaS** — managed DB service (e.g., Azure SQL Database, RDS) — provider handles OS/patching. **SaaS** — fully hosted application with embedded DB — no admin access. **IaaS** — you get a VM, you install and manage the DBMS yourself |
| **Database schema** | **Logical** — tables, relationships, constraints independent of platform. **Physical** — files, tablespaces, partitions, storage engine specifics. **View** — virtual schema presented to users/applications |
| **Data sources** | Internal systems, third-party feeds, APIs, flat files |
| **System specifications** | CPU, RAM, network bandwidth, OS requirements |

### Design Documentation

| Artifact | Purpose |
|----------|---------|
| **Data dictionary** | Catalog of every table, column, data type, constraint, and description; the "single source of truth" for schema |
| **Entity relationships (ERDs)** | Visual diagram of entities and how they relate (1:1, 1:M, M:M) |
| **Data cardinality** | Defines the numerical relationship between entities — one-to-one, one-to-many, many-to-many |
| **System requirements documentation** | Hardware, software, network, and security prerequisites for deployment |

**What they might ask:** "During planning, the team discovers the current server lacks sufficient IOPS for the projected workload — what should the DBA do first?" → Perform a gap analysis and recommend appropriate storage (SSD, SAN) or a cloud PaaS solution.
