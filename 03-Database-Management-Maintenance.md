# 3.0 Database Management and Maintenance — 25%

This is the highest-weighted domain. Expect the most questions here — invest extra study time in monitoring, optimization, change management, and documentation.

---

## 3.1 Explain the Purpose of Monitoring and Reporting

### System Alerts and Notifications

Proactive alerting prevents outages. Know what to monitor and what each metric signals.

| Alert Category | What to Watch | Why It Matters |
|----------------|---------------|----------------|
| **Growth in size / storage limits** | Database file size, tablespace usage, log file growth | Running out of disk causes writes to fail and can corrupt data |
| **Daily usage** | Query volume, active sessions, data changes per period | Establishes normal patterns; deviations signal problems |
| **Throughput** | Transactions per second (TPS), queries per second (QPS) | Declining throughput under constant load indicates a bottleneck |
| **Resource utilization — CPU** | Sustained high CPU on the DB host | May indicate missing indexes, bad query plans, or insufficient hardware |
| **Resource utilization — Memory** | Buffer pool hit ratio, page life expectancy | Low hit ratio means data is read from disk instead of cache |
| **Resource utilization — Disk space** | Free space, I/O latency, queue depth | High latency or full disks directly affect every query |
| **Resource utilization — OS performance** | Context switches, page faults, network throughput | OS-level issues cascade into DB performance |

### Baseline Configuration and Trending

- **Baseline** — a snapshot of "normal" performance metrics taken during a representative workload period.
- **Trending** — compare current metrics against the baseline over time to spot gradual degradation or growth.
- Without a baseline, you can't distinguish a genuine anomaly from normal variation.

**Exam tip:** "The DBA notices CPU usage at 80% — is this a problem?" Depends on the baseline. If the baseline is 40%, yes. If it's 78%, it's within normal range.

### Monitoring Job Completion and Failure

- Scheduled jobs (backups, index rebuilds, ETL) should send success/failure notifications.
- Failed jobs must be investigated immediately — a silently failing backup is a disaster waiting to happen.

### Replication Monitoring

- Track replication lag (seconds/transactions behind primary).
- Alert on replication errors or broken replication chains.
- Verify data consistency between primary and replicas periodically.

### Database Backup Alerts

- Confirm every scheduled backup completed successfully.
- Alert if a backup is missed, fails, or takes abnormally long.
- Validate backup file size — a dramatically smaller backup may indicate corruption or missing data.

### Transaction Log Files

- Record every data modification for recovery purposes.
- Monitor log size and growth; an unmanaged transaction log can fill the disk.
- Regular log backups (in full recovery model) truncate the log and enable point-in-time recovery.

### System Log Files

- OS-level and DBMS-level logs (error logs, event logs).
- Review for warnings, errors, and authentication failures.
- Centralize logs (syslog, SIEM) for correlation and retention.

### Deadlock Monitoring

- A **deadlock** occurs when two or more transactions each hold a lock the other needs, and neither can proceed.
- The DBMS detects deadlocks and terminates one transaction (the "victim").
- Monitor deadlock frequency and the objects involved; recurring deadlocks signal a design or coding issue.

### Connections and Sessions

| Metric | What It Tells You |
|--------|-------------------|
| **Concurrent connections** | Current active sessions; approaching the configured max means new connections will be refused |
| **Failed/attempted connections** | Authentication failures, exhausted connection pools, or possible brute-force attacks |

**What they might ask:** "The DBA sees a spike in failed connection attempts from a single IP — what should they investigate?" → Possible brute-force attack; also check if an application has stale credentials.

---

## 3.2 Explain Common Database Maintenance Processes

### Query Optimization

- Analyze execution plans (look for table scans, key lookups, sort spills).
- Rewrite inefficient queries: replace correlated subqueries with JOINs, eliminate `SELECT *`, add appropriate `WHERE` clauses.
- Use query hints sparingly and only when the optimizer consistently makes poor choices.

### Index Optimization

- **Rebuild** fragmented indexes to restore sequential page order.
- **Reorganize** lightly fragmented indexes (less resource-intensive than a rebuild).
- Remove unused indexes — every index adds overhead to `INSERT`, `UPDATE`, and `DELETE`.
- Add covering indexes for high-frequency queries to avoid key lookups.

### Patch Management

| Type | Purpose | Risk |
|------|---------|------|
| **Updates** | New features, cumulative fixes | May introduce breaking changes; test first |
| **Security patches** | Close known vulnerabilities | Delay increases exposure; apply promptly after testing |
| **Maintenance patches** | Bug fixes, stability improvements | Lower risk but still require regression testing |

**Exam tip:** The exam expects you to know that patches should be tested in a non-production environment before being applied to production, and that security patches should be prioritized.

### Database Integrity and Corruption Checks

- Run consistency checks regularly (`DBCC CHECKDB` in SQL Server, `CHECK TABLE` in MySQL, `pg_catalog` checks in PostgreSQL).
- Detect and repair page-level corruption before it propagates.
- **Table locking techniques** — integrity checks may require exclusive locks; schedule during maintenance windows to minimize user impact.

### Data Corruption Checks

- Cross-reference checksums and row counts before and after maintenance operations.
- Validate that backups restore cleanly (a corrupt backup is worse than no backup — you have false confidence).

### Periodic Review of Audit Logs

- Review who accessed what data and when.
- Identify privilege escalation, unauthorized schema changes, or unusual query patterns.
- Required for compliance frameworks (PCI DSS, GDPR, HIPAA).

### Performance Tuning

| Area | Approach |
|------|----------|
| **Transaction volumes** | Monitor TPS; tune batch sizes, commit frequency, and parallelism |
| **Load balancing** | Distribute read traffic across replicas; use connection-level or application-level routing |

### Change Management

| Element | Description |
|---------|-------------|
| **Release schedules** | Planned windows for schema changes, code deployments, and patches |
| **Capacity planning** | Project future resource needs based on trending data and business growth |
| **Upgrades** | Major version upgrades; require compatibility testing and rollback plans |
| **Vulnerability remediation** | Apply fixes for identified security issues within a defined SLA |
| **Change approval** | Formal sign-off (CAB or equivalent) before changes go to production |
| **Communication** | Notify stakeholders of upcoming changes, expected downtime, and rollback criteria |
| **Database refresh** | Copy production data to lower environments (dev, test) for realistic testing; mask sensitive data before refresh |

**Exam tip:** Change management questions often test whether you know the correct order: plan → approve → communicate → implement → validate → document.

---

## 3.3 Given a Scenario, Produce Documentation and Use Relevant Tools

This is scenario-based — expect questions like "the DBA needs to document the schema for a compliance audit — what should they produce?"

### Documentation Types

| Document | Content | When You Need It |
|----------|---------|-----------------|
| **Data dictionary** | Table names, column names, data types, constraints, descriptions, relationships | Always — the living reference for the schema |
| **Entity Relationship Diagram (ERD)** | Visual representation of entities, attributes, and relationships | Design reviews, onboarding new team members, compliance audits |
| **Maintenance documentation** | Schedules for backups, index rebuilds, patching; runbooks for common tasks | Operational continuity; handoff between DBAs |
| **SOP documentation** | Step-by-step procedures for recurring tasks | Consistency and compliance |
| **Organizational compliance documentation** | Internal policies, data-handling standards | Internal audits, governance reviews |
| **Third-party compliance documentation** | Evidence of adherence to external standards (PCI DSS, SOC 2, GDPR) | External audits and regulatory examinations |

### Tools

| Tool | Use Case |
|------|----------|
| **UML editors** (e.g., Lucidchart, draw.io, Visio, StarUML) | Create and maintain ERDs, data flow diagrams, deployment diagrams |
| **Word processors** (e.g., Word, Google Docs) | Write SOPs, maintenance docs, compliance narratives |
| **Spreadsheet tools** (e.g., Excel, Google Sheets) | Data dictionaries, capacity-planning models, audit log summaries |

**What they might ask:** "A third-party auditor requests proof that the database meets PCI DSS requirements — which documents should the DBA provide?" → Third-party compliance documentation, data dictionary (showing encryption of cardholder data), and audit log review results.

---

## 3.4 Given a Scenario, Implement Data Management Tasks

Scenario-based: you'll be given a situation and asked what action to take.

### Data Management Operations

| Task | What It Means | Decision Guidance |
|------|---------------|-------------------|
| **Modify data** | `UPDATE` existing rows | Use DML; wrap in a transaction; verify with `SELECT` before committing |
| **Define data** | `CREATE` or `ALTER` schema objects | Use DDL; follow change management; update the data dictionary |
| **Append columns** | `ALTER TABLE … ADD COLUMN` | Consider NULLability, defaults, and impact on existing queries and application code |
| **Create new data sets** | Build new tables or materialized result sets | Confirm requirements; design with proper keys and constraints |
| **Views / materialized views** | **View** — virtual, always current. **Materialized view** — physically stored snapshot, must be refreshed | Use views for real-time access; materialized views for expensive aggregations queried frequently |
| **Index creation** | `CREATE INDEX` on columns used in `WHERE`, `JOIN`, `ORDER BY` | Balance read speed vs. write overhead; don't over-index |
| **Create data tables** | `CREATE TABLE` with appropriate columns, types, constraints | Primary key, NOT NULL, CHECK constraints; foreign keys for relationships |
| **Create data relationships** | Define foreign keys between tables | Enforce referential integrity; consider cascade vs. restrict on DELETE/UPDATE |

### Data Redundancy

- **Intentional redundancy** — denormalization for read performance (e.g., storing a calculated total in a summary table).
- **Unintentional redundancy** — duplicate data without a clear purpose; leads to update anomalies.
- Know the trade-off: redundancy improves read speed but increases storage and risk of inconsistency.

### Data Sharing

- Sharing data across applications, teams, or systems (APIs, linked servers, replication, ETL).
- Security considerations: who has access, what level (read-only vs. read-write), and how credentials are managed.
- Data governance: shared data must have clear ownership and a single source of truth.

**Exam tip:** "A developer asks the DBA to create a view that joins five tables for a dashboard — but the dashboard is slow. What should the DBA consider?" → A **materialized view** with a refresh schedule, or query/index optimization on the underlying tables.
