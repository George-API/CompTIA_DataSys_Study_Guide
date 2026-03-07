# ExamObjectives ↔ ExamOutline Map (DS0-001)

Use this map to tie each study-guide section to the correct content in **ExamObjectives.md** and **ExamOutline.md**. It preserves referential integrity so the model knows exactly which objectives and outline points apply to each domain.

---

## 1.0 Database Fundamentals — 24%

**ExamObjectives.md** (primary source for depth and question types):

| Objective | Verb / task | Key topics to cover |
|-----------|-------------|----------------------|
| *(no number)* | Compare and contrast | Database structure types: relational vs. nonrelational, linear vs. non-linear; NoSQL types (document, key-value, column-oriented, graph); tools (Cassandra, MongoDB, Neo4j, Amazon DynamoDB, Cosmos) |
| 1.1 | Given a scenario, develop/modify/run | SQL: DDL, DML, set-based logic, TCLs, ACID, ANSI SQL; programming with SQL (triggers, stored procedures, functions, views) |
| 1.2 | Compare and contrast | Scripting: purpose and runtime (server vs. client); languages (PowerShell, Python); command-line (Linux, Windows) |
| 1.3 | Explain | ORM impact on DB operations: Hibernate, Entity Framework, Ebean; process to gauge impact (review ORM-generated SQL, confirm validity, impact to server, solutions) |
| 1.4 | Compare and contrast | Database planning and design: requirements gathering (users, storage, objectives); architecture (assets, gap analysis, cloud vs. on-prem, PaaS/SaaS/IaaS); schema (logical, physical, view); design docs (data dictionary, ERs, cardinality, system requirements) |

**ExamOutline.md** (high-level anchors for this domain):

- Relational vs NoSQL architectures
- SQL development: DDL, DML, TCL, ACID
- Stored procedures, views
- Scripting (Python, PowerShell, CLI)
- ORM tools (e.g., Hibernate, Entity Framework)

---

## 2.0 Database Deployment — 16%

**ExamObjectives.md**:

| Objective | Verb / task | Key topics to cover |
|-----------|-------------|----------------------|
| 2.1 | Compare and contrast | Planning and design (see 1.4 above; 2.1 is the same block in the doc — requirements, architecture, schema, design documentation) |
| 2.2 | Explain | Implementation, testing, deployment: acquisition of assets; deployment phases (install/config, provisioning, upgrading, modifying, importing); connectivity (server location, DNS, client/server, firewall, IP, ports); testing (quality check, code execution, schema, syntax, stress, triggers/alerts, version control, regression, negative); validate (index analysis, data mapping/values, queries, referential integrity, scalability) |

**ExamOutline.md**:

- Requirements gathering and architecture design
- ERDs and data dictionaries
- Schema validation, testing, version control
- Installation, configuration, provisioning, scalability validation

---

## 3.0 Database Management and Maintenance — 25%

**ExamObjectives.md**:

| Objective | Verb / task | Key topics to cover |
|-----------|-------------|----------------------|
| 3.1 | Explain | Monitoring and reporting: system alerts (growth, usage, throughput, resource utilization — CPU, memory, disk, OS); baseline/trending; job completion/failure; replication; backup alerts; transaction/system logs; deadlock; connections/sessions (concurrent, failed) |
| 3.2 | Explain | Maintenance: query/index optimization; patch management (updates, security/maintenance patches); integrity and corruption checks; table locking; audit log review; performance tuning (transaction volumes, load balancing); change management (release schedules, capacity planning, upgrades, vulnerability remediation, approval, communication); database refresh |
| 3.3 | Given a scenario | Documentation and tools: data dictionaries, ERD, maintenance docs, SOP (organizational and third-party compliance); tools (UML editors, word processors, spreadsheets) |
| 3.4 | Given a scenario | Data management tasks: modify/define data, append columns, new data sets, views/materialized views, index creation, tables, relationships; data redundancy; data sharing |

**ExamOutline.md**:

- Performance monitoring and alerting
- Query optimization and capacity planning
- Patch management and tuning
- Documentation (ERDs, SOPs, compliance artifacts)
- Data operations and relationship management

---

## 4.0 Data and Database Security — 23%

**ExamObjectives.md**:

| Objective | Verb / task | Key topics to cover |
|-----------|-------------|----------------------|
| 4.1 | Explain | Data security concepts: encryption (in transit — client-side, in-transit, server-side; at rest); data masking, discovery; destruction techniques; security audit (expired accounts, connection requests); code auditing (SQL, credential storage) |
| 4.2 | Explain | Governance and compliance: DLP, retention policies, classification (PII/PHI, PCI DSS); global/regional regulations (e.g., GDPR) |
| 4.3 | Given a scenario | Authentication and authorization: access controls (rights, privileges, least privilege); password policies; service accounts; IAM |
| 4.4 | Explain | Infrastructure security: physical (access control, biometrics, surveillance, fire suppression, cooling); logical (firewall, perimeter network, port security) |
| 4.5 | Describe | Attacks and effects: SQL injection, DoS, on-path, brute-force, phishing, malware (ransomware) |

**ExamOutline.md**:

- Encryption (in transit / at rest), masking, destruction
- Access control and identity management
- Governance and compliance (e.g., GDPR, PCI DSS)
- Infrastructure security controls
- Threat mitigation (SQL injection, DoS, ransomware, brute force)

---

## 5.0 Business Continuity — 12%

**ExamObjectives.md**:

| Objective | Verb / task | Key topics to cover |
|-----------|-------------|----------------------|
| 5.1 | Explain | Disaster recovery: DR planning; documentation (manuals, system security plan, continuity plan, build docs); techniques (replication, log shipping, high availability, mirroring); DR plan testing (RPO, RTO); transition/failback; risk assessment and continuity testing |
| 5.2 | Explain | Backup and restore: full vs. incremental (differential); database dumping; schedule/automate; test backups; validate backup hash; storage (on-site vs. off-site); retention (purge vs. archive) |

**ExamOutline.md**:

- Backup and recovery strategies
- Disaster recovery and replication
- High availability (clustering, load balancing)
- Risk assessment and continuity testing

---

## How to use this map

- **When writing a study-guide section:** For each domain, pull *all* objective rows from the table above (from ExamObjectives.md) and ensure every **ExamOutline.md** bullet for that domain is reflected. Use the “Verb / task” column to shape content (e.g., scenario steps for “Given a scenario”).
- **Depth:** Allocate space by exam weight (24%, 16%, 25%, 23%, 12%); the map does not duplicate full objective text — refer to ExamObjectives.md for exact wording and sub-bullets.
- **Referential integrity:** Nothing in the study guide should come from an objective or outline bullet that isn’t listed under that domain in this map.

---

