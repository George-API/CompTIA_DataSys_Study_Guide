# CompTIA DataSys+ DS0-001 — Terminology & Abbreviations

Quick-scan reference. All terms sourced from the official exam objectives and study guide content.

### Domain Key

| Domain | Title | Weight |
|---|---|---|
| 1 | Database Fundamentals | 24% |
| 2 | Database Deployment | 16% |
| 3 | Database Management & Maintenance | 25% |
| 4 | Data & Database Security | 23% |
| 5 | Business Continuity | 12% |

---

## Abbreviations & Acronyms

Includes the official CompTIA DS0-001 acronym list plus additional abbreviations from study content.

| Abbreviation | Full Name | Domain | Context |
|---|---|---|---|
| ACID | Atomicity, Consistency, Isolation, Durability | 1 | Transaction reliability properties; relational DBs enforce by default |
| ANSI | American National Standards Institute | 1 | ANSI SQL = vendor-neutral standard; exam favors ANSI-compliant answers |
| BASE | Basically Available, Soft state, Eventually consistent | 1 | NoSQL consistency model; contrast with ACID |
| BSON | Binary JSON | 1 | Storage format used by MongoDB |
| CAB | Change Advisory Board | 3 | Formal sign-off body for change management approval |
| CCPA | California Consumer Privacy Act | 4 | Regional US privacy regulation |
| COOP | Continuity of Operations Plan | 5 | Broader organizational plan for maintaining essential functions during disaster |
| CPU | Central Processing Unit | 3 | Monitored resource; sustained high CPU may indicate missing indexes |
| CRUD | Create, Read, Update, Delete | 1 | Basic data operations |
| CVV | Card Verification Value | 4 | PCI DSS protected data element |
| DAS | Direct-Attached Storage | 1, 2 | Storage attached directly to server; contrasted with NAS and SAN |
| DB | Database | — | General abbreviation |
| DBaaS | Database as a Service | 2 | Cloud-managed database offering |
| DBMS | Database Management System | 2 | Software that manages databases (SQL Server, MySQL, PostgreSQL, etc.) |
| DCL | Data Control Language | 1 | SQL sublanguage: `GRANT`, `REVOKE` |
| DDL | Data Definition Language | 1 | SQL sublanguage: `CREATE`, `ALTER`, `DROP`, `TRUNCATE` |
| DHCP | Dynamic Host Configuration Protocol | 2 | Dynamic IP assignment; acceptable for dev/test, not production DB servers |
| DLP | Data Loss Prevention | 4 | Policies/tools preventing unauthorized transmission of sensitive data |
| DML | Data Manipulation Language | 1 | SQL sublanguage: `SELECT`, `INSERT`, `UPDATE`, `DELETE` |
| DMZ | Demilitarized Zone | 2, 4 | Perimeter network segment; DB should **never** reside here |
| DNS | Domain Name Service | 2 | Resolves hostnames to IPs; TTL affects failover speed |
| DoS | Denial of Service | 4 | Attack that overwhelms a server to prevent legitimate access |
| DPO | Data Protection Officer | 4 | Role required under GDPR |
| DR | Disaster Recovery | 5 | Planning and techniques to restore services after catastrophic events |
| ERD | Entity Relationship Diagram | 1, 2, 3 | Visual map of entities, attributes, and relationships (1:1, 1:M, M:M) |
| ETL | Extract, Transform, Load | 1, 3 | Data integration process; commonly scheduled server-side |
| FTP | File Transfer Protocol | 2 | File transfer; know default port for security context |
| GDPR | General Data Protection Regulation | 4 | EU/EEA data protection law; right to erasure, 72-hour breach notification |
| HA | High Availability | 5 | Clustering, failover groups, always-on; near-zero RPO and RTO |
| HDD | Hard Disk Drive | 1, 2 | Traditional spinning disk; contrast with SSD |
| HIPAA | Health Insurance Portability and Accountability Act | 4 | US law governing PHI protection |
| HVAC | Heating, Ventilation, and Air Conditioning | 4 | Physical security: cooling systems for data centers |
| IaaS | Infrastructure as a Service | 1, 2 | Cloud model: you manage OS, DBMS, schema, data; provider manages hardware |
| IAM | Identity and Access Management | 4 | Centralized framework for managing identities and access rights |
| IaC | Infrastructure as Code | 2 | Automated provisioning via scripts/templates |
| IOPS | Input/Output Operations Per Second | 1, 2 | Storage performance metric; drives SSD vs HDD and SAN decisions |
| IP | Internet Protocol | 2 | Network addressing; production DBs use static IPs |
| JPA | Java Persistence API | 1 | Java ORM specification; Hibernate is the primary implementation |
| JSON | JavaScript Object Notation | 1 | Data format used by document databases |
| LDAP | Lightweight Directory Access Protocol | 4 | Directory service protocol for IAM integration |
| LUKS | Linux Unified Key Setup | 4 | Linux full-disk encryption |
| MFA | Multi-Factor Authentication | 4 | Additional authentication layer; mitigates brute-force and phishing |
| MySQL | My Structured Query Language | 2 | RDBMS; default port **3306**/TCP |
| NAS | Network-Attached Storage | 1, 2 | File-level storage over network; contrasted with DAS and SAN |
| NoSQL | Not Only SQL | 1 | Nonrelational databases: document, key-value, column-oriented, graph |
| OLAP | Online Analytical Processing | 1 | Analytical/reporting workloads; shapes schema design |
| OLTP | Online Transaction Processing | 1 | Transactional workloads; high concurrency, normalized schema |
| ORM | Object-Relational Mapping | 1 | Maps app objects to DB tables (Hibernate, Entity Framework, Ebean) |
| OS | Operating System | 2, 3 | Monitored for context switches, page faults; OS-level logs |
| PaaS | Platform as a Service | 1, 2 | Cloud model: provider manages OS/patching/DBMS; you manage schema and data |
| PCI DSS | Payment Card Industry Data Security Standard | 4 | Standard for protecting cardholder data; quarterly scans, encryption, access logging |
| PHI | Personal Health Information | 4 | Medical records, diagnoses, insurance IDs; governed by HIPAA |
| PII | Personally Identifiable Information | 4 | Name, SSN, email, address, DOB; governed by various privacy laws |
| PIPEDA | Personal Information Protection and Electronic Documents Act | 4 | Canadian federal privacy law |
| QPS | Queries Per Second | 3 | Performance throughput metric |
| RAM | Random Access Memory | 2 | System specification; affects buffer pool size and caching |
| RDBMS | Relational Database Management System | 1 | Database system using tables, rows, columns, SQL |
| REST | Representational State Transfer | — | API architectural style |
| RPO | Recovery Point Objective | 5 | Max acceptable data loss (time); drives **backup frequency** |
| RTO | Recovery Time Objective | 5 | Max acceptable downtime; drives **architecture** (HA, hot standby) |
| SaaS | Software as a Service | 1, 2 | Cloud model: provider manages everything; user has no admin access |
| SAN | Storage Area Network | 1, 2 | Block-level network storage; high performance for databases |
| SHA-256 | Secure Hash Algorithm 256-bit | 5 | Hash algorithm for validating backup integrity |
| SIEM | Security Information and Event Management | 3 | Centralized log aggregation and correlation |
| SLA | Service Level Agreement | 3 | Defines acceptable performance/availability targets |
| SNMP | Simple Network Management Protocol | 2 | Monitoring and alerting protocol |
| SOC 2 | System and Organization Controls 2 | 3, 4 | Compliance framework for service organizations |
| SOP | Standard Operating Procedure | 3 | Step-by-step documentation for recurring tasks |
| SQL | Structured Query Language | 1 | Standard language for relational databases |
| SSD | Solid State Drive | 1, 2 | Flash storage; faster IOPS than HDD |
| SSH | Secure Shell | 2 | Encrypted remote access protocol |
| SSO | Single Sign-On | 4 | One set of credentials for multiple systems; enabled by IAM |
| TCL | Transaction Control Language | 1 | SQL sublanguage: `BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT` |
| TCP/IP | Transmission Control Protocol / Internet Protocol | 2 | Core network protocol; DB connections use TCP |
| TDE | Transparent Data Encryption | 4 | Encrypts DB files at rest; queries see plaintext (decrypted in memory) |
| TLS/SSL | Transport Layer Security / Secure Sockets Layer | 4 | In-transit encryption for DB connections |
| TPS | Transactions Per Second | 3 | Performance throughput metric |
| TTL | Time To Live | 2 | DNS setting affecting failover speed |
| UML | Unified Modeling Language | 3 | Diagram standard; editors used for ERDs and data flow diagrams |
| VLAN | Virtual Local Area Network | 2 | Network segmentation |
| VPC | Virtual Private Cloud | 2 | Isolated cloud network environment |
| VXLAN | Virtual Extensible LAN | 2 | Network overlay technology |
| WAF | Web Application Firewall | 4 | Mitigation for SQL injection and web-based attacks |

---

## Key Terms & Definitions

| Term | Definition | Domain |
|---|---|---|
| **Aggregation pipeline** | MongoDB's multi-stage data processing framework for transformations and analytics | 1 |
| **Archive** | Move aged backups to long-term, low-cost storage for regulatory or legal holds | 5 |
| **Baseline** | Snapshot of "normal" performance metrics; required to distinguish anomalies from normal variation | 3 |
| **Brute-force attack** | Automated repeated login attempts; mitigated by lockout policies, MFA, strong passwords | 4 |
| **Buffer pool hit ratio** | Percentage of data reads served from memory vs disk; low ratio = performance problem | 3 |
| **Capacity planning** | Projecting future resource needs based on trending data and business growth | 3 |
| **Cardinality** | Numerical relationship between entities: one-to-one (1:1), one-to-many (1:M), many-to-many (M:M) | 1 |
| **Change management** | Lifecycle: plan → approve → communicate → implement → validate → document | 3 |
| **Client-side encryption** | Application encrypts data before sending to the server | 4 |
| **Clustering** | Multiple nodes share a workload; if one fails, another takes over (active-passive or active-active) | 5 |
| **Column-level encryption** | Encrypts specific sensitive columns; more granular than TDE but higher app complexity | 4 |
| **Column-oriented database** | NoSQL type storing data by column families; high write throughput (Cassandra, HBase) | 1 |
| **Connection pooling** | Reusing database connections to reduce overhead from establishing new connections | 1, 2 |
| **Covering index** | Index that includes all columns needed by a query, eliminating key lookups | 3 |
| **Cryptographic erasure** | Destroying the encryption key to render encrypted data permanently unreadable | 4 |
| **Cursor** | Row-by-row processing construct; almost always inferior to set-based SQL on the exam | 1 |
| **Cypher** | Query language used by Neo4j graph database | 1 |
| **Data dictionary** | Catalog of every table, column, data type, constraint, and description; living schema reference | 1, 2, 3 |
| **Data discovery** | Scanning databases to find where sensitive data (PII, PHI, PCI) resides; prerequisite to masking | 4 |
| **Data masking** | Replacing sensitive data with fictitious but realistic values; static (copies) or dynamic (query time) | 4 |
| **Database dumping** | Logical export of DB objects and data (pg_dump, mysqldump, expdp); portable, cross-platform | 5 |
| **Database refresh** | Copying production data to lower environments; requires masking sensitive data first | 3 |
| **Deadlock** | Two+ transactions each hold a lock the other needs; DBMS kills one ("victim") | 3 |
| **Denormalization** | Intentional redundancy to improve read performance; trade-off: increased storage, inconsistency risk | 3 |
| **Differential backup** | Captures all changes since the last **full** backup; restore = full + latest differential (2 files) | 5 |
| **Document database** | NoSQL type storing JSON/BSON documents with flexible schema (MongoDB, Cosmos) | 1 |
| **Dynamic masking** | Masks data at query time based on user role; production data unchanged | 4 |
| **Execution plan** | Database engine's strategy for executing a query; look for table scans, key lookups, sort spills | 3 |
| **Failback** | Process of returning operations from DR site to the restored primary site | 5 |
| **Failover** | Automatic or manual switch to a standby system when the primary fails | 5 |
| **Foreign key** | Constraint enforcing referential integrity between tables; consider cascade vs restrict on DELETE | 1, 3 |
| **Full backup** | Captures the entire database; fastest restore (1 file), largest storage | 5 |
| **Full-disk encryption** | OS-level encryption of the entire volume (BitLocker, LUKS) | 4 |
| **Function** | SQL object returning a scalar value or table; usable inside SELECT statements | 1 |
| **Gap analysis** | Comparing current assets against requirements to identify what's missing | 1, 2 |
| **Graph database** | NoSQL type using nodes + edges; optimized for relationship traversal (Neo4j) | 1 |
| **Gremlin** | Graph traversal query language; used by Cosmos DB's graph API | 1 |
| **Incremental backup** | Captures changes since last backup **of any type**; restore = full + every incremental in order (N files) | 5 |
| **In-transit encryption** | Encrypting the transport channel itself (TLS/SSL between app and DB) | 4 |
| **Key-value store** | NoSQL type mapping unique keys to blobs/values; fast lookups (DynamoDB, Redis) | 1 |
| **Least privilege** | Grant only the permissions required for a user's job; default deny | 4 |
| **Load balancing** | Distributing traffic across replicas or nodes to prevent bottlenecks | 3, 5 |
| **Log shipping** | Periodically shipping transaction log backups to a standby server | 5 |
| **Logical deletion** | Soft delete; marks records as deleted but data remains recoverable | 4 |
| **Logical schema** | Platform-independent: entities, attributes, relationships, constraints | 1, 2 |
| **Materialized view** | Physically stored query result snapshot; stale until refreshed; best for expensive aggregations | 1, 3 |
| **Media sanitization** | Degaussing, overwriting, or physical destruction of storage media | 4 |
| **Mirroring** | Real-time database copy to a mirror server; synchronous or asynchronous | 5 |
| **N+1 query problem** | ORM issues one query per related object instead of a single JOIN; major performance anti-pattern | 1 |
| **Negative testing** | Deliberately sending bad data to confirm the system handles errors gracefully | 2 |
| **On-path attack** | Man-in-the-middle; attacker intercepts client↔server communication; mitigated by TLS | 4 |
| **Over-fetching** | Selecting all columns (SELECT *) when only a few are needed; ORM anti-pattern | 1 |
| **Page life expectancy** | How long data pages stay in the buffer pool; low values indicate memory pressure | 3 |
| **Parameterized queries** | SQL with placeholders for user input; primary defense against SQL injection | 4 |
| **Patch management** | Updates, security patches, maintenance patches; always test in non-production first | 3 |
| **Perimeter network** | DMZ; web servers sit here, databases do **not** | 4 |
| **Phishing** | Social engineering via fake emails/websites to steal credentials; mitigated by MFA, training | 4 |
| **Physical deletion** | DELETE or TRUNCATE; data may still be recoverable from backups or forensics | 4 |
| **Physical schema** | Platform-specific: tables, indexes, partitions, tablespaces, storage engine | 1, 2 |
| **Port security** | Restricting open ports; changing defaults; disabling unused services | 4 |
| **Purge** | Delete backups after retention period expires; routine operational strategy | 5 |
| **Ransomware** | Malware that encrypts DB files and demands payment; mitigated by offline/immutable backups | 4 |
| **Referential integrity** | Foreign key enforcement ensuring no orphan records exist | 2, 3 |
| **Regression testing** | Verifying existing functionality still works after changes | 2 |
| **Replication** | Continuously copying data changes to standby servers; synchronous or asynchronous | 3, 5 |
| **Replication lag** | Time/transactions a replica is behind the primary; critical monitoring metric | 3 |
| **Risk assessment** | Identifying threats, their likelihood, and impact on database availability | 5 |
| **Schema-on-read** | Structure interpreted at query time; NoSQL pattern | 1 |
| **Schema-on-write** | Schema defined before data entry; relational pattern | 1 |
| **Server-side encryption** | Server encrypts data upon receipt before storing; common in cloud providers | 4 |
| **Service account** | Dedicated account for applications/automation; minimal privileges, rotate passwords, secrets manager | 4 |
| **SQL injection** | Attacker inserts malicious SQL via input fields; primary fix = parameterized queries | 4 |
| **Static masking** | Permanently alters data in non-production copies; production data unchanged | 4 |
| **Stored procedure** | Precompiled SQL routine accepting parameters; reduces round-trips, encapsulates logic | 1 |
| **Stress testing** | Sustained high call volume against stored procedures or simulated user load against the full stack | 2 |
| **Tabletop exercise** | Walk-through of DR plan on paper; low cost, catches procedural gaps | 5 |
| **3-2-1 rule** | 3 backup copies, 2 different media types, 1 off-site | 5 |
| **Trending** | Comparing current metrics against the baseline over time to spot degradation or growth | 3 |
| **Trigger** | SQL that fires automatically on INSERT, UPDATE, or DELETE events; used for audit logging | 1 |
| **TRUNCATE** | DDL statement removing all rows; minimal logging, cannot use WHERE, keeps table structure | 1 |
| **View** | Virtual table defined by a SELECT; no data stored; always current | 1, 3 |
| **View schema** | Virtual tables exposing curated subsets of physical schema data | 1, 2 |
| **Vulnerability remediation** | Applying fixes for identified security issues within a defined SLA | 3 |

---

## Default Database Ports

| DBMS | Default Port |
|---|---|
| SQL Server | **1433**/TCP |
| MySQL | **3306**/TCP |
| PostgreSQL | **5432**/TCP |
| MongoDB | **27017**/TCP |

---

## SQL Statements by Sublanguage

| Sublanguage | Statements | Controls |
|---|---|---|
| **DDL** | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` | Schema structure |
| **DML** | `SELECT`, `INSERT`, `UPDATE`, `DELETE` | Data within tables |
| **DCL** | `GRANT`, `REVOKE` | Access permissions |
| **TCL** | `BEGIN TRANSACTION`, `COMMIT`, `ROLLBACK`, `SAVEPOINT` | Transactions |

---

## NoSQL Tools Quick Reference

| Tool | Type | Key Fact |
|---|---|---|
| **Cassandra** | Column-oriented | Apache; masterless ring; linear scalability |
| **MongoDB** | Document | BSON; aggregation pipeline; flexible schema |
| **Neo4j** | Graph | Cypher query language; ACID-compliant |
| **DynamoDB** | Key-value | AWS managed; single-digit ms latency |
| **Cosmos DB** | Multi-model | Azure; document, key-value, graph, column-family APIs |
| **Redis** | Key-value | In-memory; caching and session store |
| **HBase** | Column-oriented | Hadoop ecosystem; wide-column store |

---

## ORM Tools

| ORM | Language | Key Fact |
|---|---|---|
| **Hibernate** | Java / JPA | Mature; lazy/eager loading; generates complex SQL |
| **Entity Framework** | .NET (C#) | Code-First and Database-First workflows |
| **Ebean** | Java | Lightweight alternative to Hibernate |

---

## Compliance & Regulation Quick Reference

| Standard / Law | Scope | Key Requirement |
|---|---|---|
| **PCI DSS** | Cardholder data | Encryption, network segmentation, quarterly scans, access logging |
| **HIPAA** | PHI (US healthcare) | Minimum necessary access, audit trails, safeguards |
| **GDPR** | EU/EEA residents' data | Right to erasure, data portability, 72-hour breach notification, DPO |
| **CCPA** | California residents | Consumer data rights, opt-out of sale |
| **SOC 2** | Service organizations | Trust service criteria: security, availability, processing integrity |
