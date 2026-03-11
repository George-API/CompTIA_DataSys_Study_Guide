# Cross-Domain Review

Use this after reading domains 01-05 and before taking the practice questions. It consolidates the connections, distinctions, and sequences that span multiple domains - the seams the exam targets.

---

## Concept Threads Across Domains

Concepts that appear in more than one domain. The exam can ask about any of these from multiple angles.

| Concept | How the Angle Shifts |
|--------|----------------------|
| **ERDs** | Design: validate logical schema. Deployment: confirm before build. Maintenance: keep current for audits |
| **Data dictionary** | Design: define schema. Deployment: reference during build. Maintenance: update when schema changes |
| **Schema (logical / physical / view)** | Planning: compare levels. Deployment: build physical from logical. Testing: validate schema meets requirements |
| **Indexes** | Fundamentals: SQL syntax. Deployment: validate coverage. Maintenance: rebuild/reorganize. Data management: create new ones |
| **Views / materialized views** | Fundamentals: what they are. Maintenance: when to use which (real-time vs. expensive aggregation) |
| **Stored procedures** | Fundamentals: encapsulate logic. Deployment: stress test them. Maintenance: optimize execution |
| **Triggers** | Fundamentals: automatic event response. Deployment: verify they fire correctly |
| **Encryption** | Security: protect live data. Continuity: encrypt backups and retain keys for the same duration |
| **Audit logs** | Monitoring: collect them. Maintenance: review periodically. Security: check for anomalies. Compliance: retain per regulation |
| **Backup** | Monitoring: alert on failure. Maintenance: validate integrity. DR: drives RPO. Backup: types, scheduling, testing |
| **Replication** | Monitoring: track lag and errors. DR: synchronous vs. asynchronous trade-offs |
| **Load balancing** | Maintenance: distribute read traffic. DR: part of high-availability architecture |
| **Firewall / perimeter network** | Deployment: open correct ports between tiers. Security: DB never in the DMZ |
| **Change management** | Deployment: approval before modifying config. Maintenance: full lifecycle (plan through document) |
| **Service accounts** | Deployment: provision before install. Security: least privilege, rotate passwords, secrets manager |
| **Compliance documentation** | Maintenance: produce and maintain docs. Security: meet regulatory requirements |
| **Data masking** | Maintenance: mask sensitive data before refreshing lower environments. Security: static vs. dynamic masking |
| **Retention** | Governance: define how long to keep data. Backup: implement purge or archive cycles |
---

## Key Distinctions

The pairs and groups the exam tests through close distractors. If you can't instantly distinguish both sides, review the referenced section.

### SQL Sublanguages (1.1)

| Sublanguage | Controls | Key Statements |
|-------------|----------|----------------|
| **DDL** | Schema structure | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` |
| **DML** | Data within tables | `SELECT`, `INSERT`, `UPDATE`, `DELETE` |
| **DCL** | Access permissions | `GRANT`, `REVOKE` |
| **TCL** | Transactions | `BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT` |

Trap: `TRUNCATE` is DDL (not DML) even though it removes data. `DELETE` is DML.

### Trigger vs. Stored Procedure (1.1)

| | Trigger | Stored Procedure |
|-|---------|-----------------|
| **Invocation** | Automatic - fires on INSERT/UPDATE/DELETE | Manual - must be called by name |
| **Use case** | Audit logging, enforcing rules on every change | Reusable business logic called from applications |

Trap: "automatically log every UPDATE" = trigger. "Application calls a reusable routine" = stored procedure.

### View vs. Materialized View (1.1 / 3.4)

| | View | Materialized View |
|-|------|-------------------|
| **Storage** | No data stored - runs the SELECT every time | Physically stored snapshot |
| **Freshness** | Always current | Stale until refreshed |
| **Best for** | Simplifying access, restricting columns | Expensive aggregations queried often |

### Schema Levels (1.4 / 2.1)

| Level | Platform-dependent? | Contains |
|-------|-------------------|----------|
| **Logical** | No | Entities, attributes, relationships, constraints |
| **Physical** | Yes | Tables, indexes, partitions, tablespaces, storage engine |
| **View** | Yes | Virtual tables exposing subsets of physical schema |

### Cloud Hosting Models (1.4 / 2.1)

| Model | You Manage | Provider Manages |
|-------|-----------|-----------------|
| **IaaS** | OS, DBMS, schema, data | Hardware, network, virtualization |
| **PaaS** | Schema, data | Hardware, OS, DBMS patching |
| **SaaS** | Nothing (user only) | Everything |

### Static vs. Dynamic Masking (4.1)

| | Static | Dynamic |
|-|--------|---------|
| **When** | Before data leaves production (copy) | At query time in production |
| **Production data** | Unchanged (mask applied to copy) | Unchanged (mask applied to results) |
| **Use case** | Safe test/dev environments | Role-based access in production |

### PII vs. PHI vs. PCI DSS (4.2)

| Classification | Covers | Governed By |
|----------------|--------|------------|
| **PII** | Name, SSN, email, address, DOB | Various (state/federal privacy laws) |
| **PHI** | Medical records, diagnoses, insurance IDs | HIPAA |
| **PCI DSS** | Card number, CVV, cardholder name, expiration | PCI SSC |

Trap: "medical diagnosis" = PHI. "credit card number" = PCI DSS. "Social Security number" = PII.

### Backup Types (5.2)

| Type | Captures changes since | Restore requires |
|------|----------------------|-----------------|
| **Full** | N/A (entire database) | Full only |
| **Differential** | Last **full** backup | Full + latest differential |
| **Incremental** | Last backup of **any type** | Full + every incremental in order |

Trap: Differential grows over time but restore is simpler (2 files). Incremental files are small but restore is slowest (N files in sequence).

### RPO vs. RTO (5.1)

| Metric | Question it answers | Drives |
|--------|-------------------|--------|
| **RPO** | "How much data can we lose?" | Backup frequency |
| **RTO** | "How fast must we recover?" | Architecture (HA, hot standby, cold restore) |

### DR Techniques (5.1)

| Technique | RPO | RTO | Trade-off |
|-----------|-----|-----|-----------|
| **Synchronous replication/mirroring** | Near-zero | Seconds | Adds write latency (waits for ack) |
| **Asynchronous replication/mirroring** | Seconds-minutes | Seconds-minutes | Faster writes, risk of recent data loss |
| **Log shipping** | Minutes-hours | Minutes-hours | Cheapest, highest data-loss risk |
| **HA clustering** | Near-zero | Near-zero | Most complex and expensive |

---

## Ordered Processes

Sequences the exam tests by asking "what should the DBA do FIRST?" or "what is the correct order?"

### ORM Impact Assessment (1.3)

1. Review SQL code generated by the ORM
2. Confirm validity of code
3. Determine impact to the database server
4. Provide solutions / alternate approach

First step is always **review the generated SQL**.

### Deployment Phases (2.2)

1. Acquisition of assets
2. Installation and configuration (verify prerequisites)
3. Provisioning (allocate storage, create schemas)
4. Importing (load data, validate)
5. Testing and validation

### Change Management (3.2)

1. Plan
2. Approve (CAB sign-off)
3. Communicate (stakeholders, downtime, rollback criteria)
4. Implement
5. Validate
6. Document

Approval comes **before** implementation. Communication comes **before** implementation.

### Failback to Normal Operations (5.1)

1. Verify primary environment is restored and healthy
2. Resynchronize data from DR site to primary
3. Validate data integrity (row counts, checksums, referential integrity)
4. Switch traffic back to primary (DNS, connection strings, load balancer)
5. Monitor for stability
6. Update DR documentation with lessons learned

### Security Audit Checklist (4.1 / 4.3)

1. Check for expired/dormant accounts
2. Review connection request logs for anomalies
3. Audit SQL code for injection vulnerabilities
4. Verify credential storage (no plaintext, no hardcoded strings)
5. Confirm password policies are enforced
6. Validate least-privilege on all service accounts

---

## Domain Weight Distribution

Allocate study time proportionally. Domains 1, 3, and 4 make up 72% of the exam.

```
Domain 3  ████████████████████████████████  25%
Domain 1  █████████████████████████████     24%
Domain 4  ████████████████████████████      23%
Domain 2  ████████████████████             16%
Domain 5  ██████████████                   12%
```

**Study order:** 01 → 02 → 03 → 04 → 05 → this file → 00-Practice-Questions

