# 2.0 Database Deployment — 16%

---

## 2.1 Compare and Contrast Aspects of Database Planning and Design

Planning and design spans both Domains 1 and 2. This section focuses on how planning feeds directly into deployment decisions.

### Requirements Gathering

| Requirement | Key Questions | Deployment Impact |
|-------------|---------------|-------------------|
| **Number of users** | How many concurrent connections at peak? | Determines connection pooling, licensing tier, and whether clustering is needed |
| **Storage capacity — Size** | How much data at launch? Growth rate? | Drives initial disk allocation, auto-growth settings, and storage tier (SSD vs. HDD) |
| **Storage capacity — Speed** | What IOPS and latency are acceptable? | Influences choice of DAS, NAS, or SAN; SSD vs. spinning disk |
| **Storage capacity — Type** | Block, file, or object storage? | Affects backup strategy and cloud service selection |
| **Database objectives / Use cases** | OLTP? OLAP? Mixed? Reporting replica? | Shapes schema normalization level, indexing strategy, and read-replica topology |

### Architecture Decisions

| Decision | Options | Trade-offs |
|----------|---------|------------|
| **Cloud vs. on-premises** | Cloud: elastic, managed, OpEx. On-prem: full control, CapEx, data-sovereignty compliance | Cloud reduces operational burden but may introduce latency or regulatory concerns |
| **Cloud hosting model** | **IaaS** — full OS control, you manage DBMS. **PaaS** — provider manages OS/patching, you manage schema and data. **SaaS** — no admin access, embedded DB | Higher abstraction = less control but less operational overhead |
| **Gap analysis** | Inventory current assets → identify gaps in hardware, licensing, skills | Drives procurement and training before deployment |

### Schema Levels

| Level | What It Represents | Created When |
|-------|--------------------|--------------|
| **Logical schema** | Entities, attributes, relationships, constraints — platform-independent | During design (ERD phase) |
| **Physical schema** | Tables, indexes, partitions, tablespaces, storage engine — platform-specific | During implementation, derived from logical schema |
| **View schema** | Virtual tables exposing curated subsets of data | After physical schema is in place; used for access control and simplification |

### Design Documentation

| Document | Purpose | When It's Used |
|----------|---------|----------------|
| **Data dictionary** | Defines every column, data type, constraint, default, and description | Referenced throughout deployment and maintenance |
| **Entity Relationship Diagram (ERD)** | Visual map of entities, attributes, and relationships (1:1, 1:M, M:M) | Validates logical schema before physical build |
| **Data cardinality** | Specifies relationship ratios between entities | Ensures correct foreign-key design and JOIN expectations |
| **System requirements documentation** | CPU, RAM, OS, network, licensing prerequisites | Procurement checklist before installation |

**Exam tip:** If a question asks "what should the DBA review before deploying a new database?", the answer set will include ERDs, data dictionaries, and system requirements — not monitoring dashboards (that's Domain 3).

---

## 2.2 Explain Database Implementation, Testing, and Deployment

### Acquisition of Assets

Before deployment begins, confirm all required assets are in hand:

- Hardware (servers, storage, network equipment) or cloud subscriptions
- Software licenses (DBMS, monitoring tools, backup agents)
- Network allocations (IPs, DNS entries, firewall rules)
- Credentials and service accounts

### Phases of Deployment

| Phase | What Happens | Key Considerations |
|-------|--------------|-------------------|
| **Installation and configuration** | Install DBMS binaries; set instance parameters (memory, tempdb, ports) | Verify **database prerequisites** — OS version, patches, runtime libraries |
| **Provisioning** | Allocate storage, create databases, set up schemas and users | Automate where possible (scripts, IaC) for repeatability |
| **Upgrading** | Move from an older DBMS version to a newer one | Compatibility checks, deprecated-feature audit, rollback plan |
| **Modifying** | Alter existing configuration (add filegroups, change collation, resize) | Change management approval; test in non-production first |
| **Importing** | Load initial data from CSVs, flat files, or another database | Validate row counts, data types, encoding; disable indexes during bulk load for speed |

### Database Connectivity

| Component | Details |
|-----------|---------|
| **Database server location** | On-prem data center, cloud region, or hybrid; latency to application tier matters |
| **DNS** | Clients resolve the DB hostname via DNS; CNAME or A records; TTL affects failover speed |
| **Client/server architecture** | Application connects to a listener on the DB server over TCP/IP |
| **Firewall and perimeter network** | Only required ports open between app tier and DB tier; DB should never be directly internet-facing |
| **Static vs. dynamic IP addressing** | Production DB servers typically use **static IPs** for predictable firewall rules and DNS; DHCP is acceptable for dev/test |
| **Ports/protocols** | SQL Server: 1433/TCP. MySQL: 3306/TCP. PostgreSQL: 5432/TCP. MongoDB: 27017/TCP. Know defaults and how to change them for security |

### Testing

| Test Type | What You're Validating |
|-----------|----------------------|
| **Database quality check** | Columns, tables, and fields match the design spec |
| **Code execution** | Stored procedures, functions, triggers run without errors |
| **Schema meets original requirements** | Logical schema → physical schema mapping is correct; all entities present |
| **Syntax errors** | SQL scripts parse and execute cleanly |
| **Stress testing** | Stored-procedure stress test: sustained high call volume. Application stress test: simulated user load against the full stack |
| **Notification triggers and alerts** | Verify that alerting (email, SNMP, webhook) fires correctly on thresholds |
| **Version control testing** | Schema scripts are checked in; migrations apply cleanly from a clean state |
| **Regression testing** | Existing functionality still works after changes |
| **Negative testing** | Invalid inputs, constraint violations, and edge cases are rejected gracefully |

**Exam tip:** Negative testing is often a distractor if you're unfamiliar with it. It deliberately sends bad data to confirm the system handles errors properly — it's not about testing for performance problems.

### Validation

| Validation Task | Purpose |
|-----------------|---------|
| **Index analysis** | Confirm indexes exist for frequent query patterns; no duplicate or unused indexes |
| **Data mapping** | Source-to-target field mapping is accurate after import |
| **Data values** | Spot-check actual data against expected values; look for NULLs, truncation, encoding issues |
| **Queries** | Key queries return correct results within acceptable time |
| **Referential integrity / integrity validation** | Foreign keys enforced; orphan records do not exist |
| **Scalability validation** | System handles projected growth — increase data volume and user count to verify |

**What they might ask:** "After importing data from a CSV, the DBA notices some columns contain unexpected NULLs — which validation step was missed?" → **Data values** validation (and possibly data mapping).
