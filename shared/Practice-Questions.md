# CompTIA DataSys+ DS0-001 Practice Questions

Cross-domain, interleaved question bank organized by CompTIA verb type. Questions are deliberately mixed across domains to force genuine retrieval rather than pattern-matching within a single topic.

**How to use:** Read the question, commit to an answer, then expand the answer. If you got it wrong, go to the referenced study guide section before continuing.

---

## Compare and Contrast

**Q1.** An organization needs a database to store semi-structured product catalog data where each product category has different attributes. The schema is expected to change frequently as new product types are added. Which database type is BEST suited?

A. Relational database with strict normalization
B. Column-oriented database
C. Document database
D. Graph database

<details><summary>Answer</summary>

**C.** Document databases store JSON/BSON documents where each document can have a different structure, making them ideal for semi-structured data with variable attributes. A is wrong because relational databases use fixed schemas that resist frequent changes. B is optimized for analytics and high write throughput, not flexible schemas. D is optimized for relationship traversal, not variable document structures.

**Domain:** 1.0 | **Objective:** (none) | **Verb:** Compare and contrast

</details>

---

**Q2.** A social media company needs to model users, their connections, and shared interests to power a recommendation engine. Which NoSQL type and tool is the BEST fit?

A. Key-value - Amazon DynamoDB
B. Document - MongoDB
C. Column-oriented - Cassandra
D. Graph - Neo4j

<details><summary>Answer</summary>

**D.** Graph databases are optimized for relationship traversal - nodes, edges, and properties map directly to users, connections, and shared attributes. Neo4j uses the Cypher query language for efficient graph queries. A is optimized for fast key lookups. B handles flexible documents but not relationship-heavy traversal. C excels at high write throughput for time-series, not relationship modeling.

**Domain:** 1.0 | **Objective:** (none) | **Verb:** Compare and contrast

</details>

---

**Q3.** A company is evaluating NoSQL tools. They need a fully managed cloud service that can support document, key-value, graph, AND column-family workloads from a single platform. Which tool meets this requirement?

A. Cassandra
B. MongoDB
C. Neo4j
D. Cosmos

<details><summary>Answer</summary>

**D.** Azure Cosmos DB is a multi-model database that supports document, key-value, graph (Gremlin API), and column-family APIs from a single service. A is column-oriented only. B is document-only. C is graph-only.

**Domain:** 1.0 | **Objective:** (none) | **Verb:** Compare and contrast

</details>

---

**Q4.** How does a relational database primarily differ from a NoSQL database in terms of scaling?

A. Relational scales horizontally; NoSQL scales vertically
B. Relational scales vertically; NoSQL scales horizontally
C. Both scale horizontally by default
D. Both scale vertically by default

<details><summary>Answer</summary>

**B.** Relational databases typically scale vertically (bigger server - more CPU, RAM, disk). NoSQL databases are designed for horizontal scaling (adding more nodes). A reverses the relationship. C and D are incorrect because the default scaling strategy differs between the two types.

**Domain:** 1.0 | **Objective:** (none) | **Verb:** Compare and contrast

</details>

---

**Q5.** A DBA needs to automate a nightly database backup on a Linux server. Which combination of tool and scheduling mechanism should they use?

A. PowerShell and Task Scheduler
B. `sqlcmd` and Task Scheduler
C. `pg_dump` and `cron`
D. `Invoke-Sqlcmd` and SQL Server Agent

<details><summary>Answer</summary>

**C.** On Linux, `pg_dump` (or `mysqldump`) handles the backup and `cron` schedules it. A and B use Windows-specific tools. D uses PowerShell and SQL Server Agent, which are Windows-ecosystem tools.

**Domain:** 1.0 | **Objective:** 1.2 | **Verb:** Compare and contrast

</details>

---

**Q6.** Which scripting language is BEST suited for a DBA who needs to automate SQL Server administration tasks on Windows, including Active Directory integration?

A. Python
B. Bash
C. PowerShell
D. PHP

<details><summary>Answer</summary>

**C.** PowerShell has native Windows automation with rich cmdlets for SQL Server (`Invoke-Sqlcmd`) and Active Directory. A is cross-platform and capable but lacks native Windows/AD integration. B is a Linux shell. D is a web development language, not a DB administration tool.

**Domain:** 1.0 | **Objective:** 1.2 | **Verb:** Compare and contrast

</details>

---

**Q7.** A DBA is choosing between running scripts server-side vs. client-side. Which is an advantage of server-side execution?

A. Users can run ad-hoc queries without server access
B. Credentials are more easily managed in the browser
C. Scripts leverage server resources and stay closer to the data
D. No OS scheduler is required for automation

<details><summary>Answer</summary>

**C.** Server-side scripts run on the database or application server, leveraging server resources with minimal network data transfer. A describes client-side use. B is incorrect - client-side risks exposing credentials. D is wrong - server-side automation typically uses OS schedulers like `cron` or Task Scheduler.

**Domain:** 1.0 | **Objective:** 1.2 | **Verb:** Compare and contrast

</details>

---

**Q8.** Which SQL sublanguage does the `TRUNCATE` statement belong to?

A. DML
B. DDL
C. TCL
D. DCL

<details><summary>Answer</summary>

**B.** `TRUNCATE` is DDL (Data Definition Language). Although it removes data, it operates on the table structure (minimal logging, cannot use `WHERE`, resets identity). `DELETE` is DML. A is wrong - DML includes `SELECT`, `INSERT`, `UPDATE`, `DELETE`. C controls transactions. D controls permissions (`GRANT`, `REVOKE`).

**Domain:** 1.0 | **Objective:** 1.1 | **Verb:** Compare and contrast

</details>

---

**Q9.** What is the key difference between a logical schema and a physical schema?

A. A logical schema includes indexes and partitions; a physical schema does not
B. A physical schema is platform-independent; a logical schema is platform-specific
C. A logical schema defines entities and relationships independent of platform; a physical schema specifies storage-engine details
D. A view schema and a logical schema are the same thing

<details><summary>Answer</summary>

**C.** The logical schema defines entities, attributes, relationships, and constraints without tying to a specific platform. The physical schema maps this to actual tables, indexes, partitions, tablespaces, and storage engine specifics. A and B reverse the relationship. D is wrong - a view schema is a virtual layer on top of the physical schema.

**Domain:** 1.0 / 2.0 | **Objective:** 1.4 / 2.1 | **Verb:** Compare and contrast

</details>

---

**Q10.** An organization wants a cloud-hosted database where the provider manages the operating system and patching, but the DBA retains control over the schema and data. Which cloud model fits?

A. IaaS
B. PaaS
C. SaaS
D. On-premises

<details><summary>Answer</summary>

**B.** PaaS (Platform as a Service) - the provider manages the OS and patching; the DBA manages the schema and data (e.g., Azure SQL Database, Amazon RDS). A gives you a VM where you manage everything including the OS. C provides a fully hosted application with no DBA access. D is not a cloud model.

**Domain:** 1.0 / 2.0 | **Objective:** 1.4 / 2.1 | **Verb:** Compare and contrast

</details>

---

**Q11.** Which design document provides a catalog of every table, column, data type, constraint, and description in the database?

A. Entity Relationship Diagram
B. System requirements documentation
C. Data dictionary
D. Standard operating procedure

<details><summary>Answer</summary>

**C.** The data dictionary is the "single source of truth" for schema metadata. A shows visual relationships between entities but not column-level detail. B covers hardware and software prerequisites. D documents recurring operational procedures.

**Domain:** 1.0 / 2.0 | **Objective:** 1.4 / 2.1 | **Verb:** Compare and contrast

</details>

---

## Given a Scenario

**Q12.** A DBA needs to remove all rows from a staging table before a nightly data load. The table structure must be preserved, and the operation should complete as quickly as possible with minimal transaction log usage. Which statement should the DBA use?

A. `DROP TABLE staging;`
B. `DELETE FROM staging;`
C. `TRUNCATE TABLE staging;`
D. `ALTER TABLE staging;`

<details><summary>Answer</summary>

**C.** `TRUNCATE` removes all rows with minimal logging and preserves the table structure. A drops the entire table (structure gone). B deletes all rows but is fully logged and slower. D modifies the table structure, not data.

**Domain:** 1.0 | **Objective:** 1.1 | **Verb:** Given a scenario

</details>

---

**Q13.** The DBA wants to automatically record every `DELETE` operation on the `orders` table into an `orders_audit` table. Which SQL object should they create?

A. View
B. Stored procedure
C. Function
D. Trigger

<details><summary>Answer</summary>

**D.** A trigger fires automatically in response to `INSERT`, `UPDATE`, or `DELETE` events - no application changes needed. A is a virtual table, not an event handler. B must be called explicitly. C returns values within queries, not event-driven.

**Domain:** 1.0 | **Objective:** 1.1 | **Verb:** Given a scenario

</details>

---

**Q14.** A DBA is transferring funds between two bank accounts. If either update fails, neither should be applied. Which approach ensures this?

A. Run two separate `UPDATE` statements
B. Use a `VIEW` to combine both accounts
C. Wrap both updates in `BEGIN TRANSACTION` ... `COMMIT` with a `ROLLBACK` on error
D. Use `TRUNCATE` to reset both accounts

<details><summary>Answer</summary>

**C.** TCL (Transaction Control Language) ensures atomicity - either both updates succeed (`COMMIT`) or neither does (`ROLLBACK`). A provides no atomicity guarantee. B is a virtual table, not a transaction mechanism. D destroys all data in the tables.

**Domain:** 1.0 | **Objective:** 1.1 | **Verb:** Given a scenario

</details>

---

**Q15.** An application reports slow database queries after upgrading the ORM framework from Hibernate 5 to Hibernate 6. What should the DBA do FIRST?

A. Add more RAM to the database server
B. Rewrite all queries as stored procedures
C. Review the SQL code generated by the ORM
D. Create new indexes on all tables

<details><summary>Answer</summary>

**C.** The exam-critical sequence for ORM impact starts with reviewing the generated SQL. Until the DBA sees what SQL the ORM is actually producing, any other action is premature. A addresses a symptom without diagnosis. B is a major refactor, not a first step. D may help but without knowing which queries are slow, you'd be guessing.

**Domain:** 1.0 | **Objective:** 1.3 | **Verb:** Given a scenario

</details>

---

**Q16.** After importing customer data from a CSV file, the DBA notices that several date columns contain unexpected NULL values. Which validation step was MOST likely missed?

A. Index analysis
B. Scalability validation
C. Data values validation
D. Regression testing

<details><summary>Answer</summary>

**C.** Data values validation involves spot-checking actual data against expected values, looking for NULLs, truncation, and encoding issues. A checks index coverage. B tests growth handling. D verifies existing functionality after changes - not import accuracy.

**Domain:** 2.0 | **Objective:** 2.2 | **Verb:** Given a scenario

</details>

---

**Q17.** During deployment testing, the QA team submits intentionally malformed inputs such as special characters, excessively long strings, and SQL injection patterns. Which type of testing is this?

A. Stress testing
B. Regression testing
C. Negative testing
D. Version control testing

<details><summary>Answer</summary>

**C.** Negative testing deliberately sends invalid inputs to confirm the system rejects them gracefully. A tests performance under load. B verifies existing features still work after changes. D verifies schema scripts apply cleanly from source control.

**Domain:** 2.0 | **Objective:** 2.2 | **Verb:** Given a scenario

</details>

---

**Q18.** A third-party auditor requests evidence that the database meets PCI DSS requirements. Which documents should the DBA provide?

A. Maintenance documentation and UML diagrams
B. Third-party compliance documentation, data dictionary, and audit log review results
C. Entity Relationship Diagram and system requirements documentation only
D. Standard operating procedures only

<details><summary>Answer</summary>

**B.** PCI DSS is an external standard, so third-party compliance documentation is required. The data dictionary shows encryption of cardholder data fields, and audit log reviews demonstrate access monitoring. A is for operational continuity, not compliance evidence. C lacks compliance-specific artifacts. D covers procedures but not compliance proof.

**Domain:** 3.0 | **Objective:** 3.3 | **Verb:** Given a scenario

</details>

---

**Q19.** A developer requests a real-time view for a dashboard that joins five tables. The dashboard queries are slow. What should the DBA consider?

A. Delete the view and create a stored procedure
B. Create a materialized view with a refresh schedule
C. Add a trigger to pre-populate the dashboard
D. Move the database to a larger server

<details><summary>Answer</summary>

**B.** A materialized view physically stores the query result and can be refreshed on a schedule, eliminating repeated expensive JOINs. A changes the access pattern but doesn't solve the join cost. C adds complexity without addressing the core performance issue. D might help but doesn't address the query design.

**Domain:** 3.0 | **Objective:** 3.4 | **Verb:** Given a scenario

</details>

---

**Q20.** A DBA needs to add a `region` column to an existing `customers` table that has 10 million rows. Which consideration is MOST important?

A. Whether the table has any indexes
B. The NULLability and default value for the new column
C. The number of views referencing the table
D. Whether the table uses ACID transactions

<details><summary>Answer</summary>

**B.** When appending a column with `ALTER TABLE ... ADD COLUMN`, the DBA must decide on NULLability (allow NULLs or define a default for existing rows) and assess impact on existing queries and application code. A is relevant but secondary. C may matter for downstream queries but isn't the primary concern during the ALTER. D is a property of the DBMS, not the ALTER operation.

**Domain:** 3.0 | **Objective:** 3.4 | **Verb:** Given a scenario

</details>

---

**Q21.** A new data analyst needs to run reports against the production database but should not be able to modify any data. What should the DBA do?

A. Grant the analyst `db_owner` permissions with a verbal agreement not to modify data
B. Create a read-only role with `SELECT` permissions and assign the analyst to it
C. Give the analyst the application's service account credentials
D. Grant `INSERT`, `UPDATE`, `DELETE`, and `SELECT` permissions, then revoke the write permissions later

<details><summary>Answer</summary>

**B.** Least privilege: create a role scoped to exactly what's needed (`SELECT` only) and assign the user. A violates least privilege. C shares a service account (security violation). D grants excessive permissions first - the correct approach is to start with minimum access.

**Domain:** 4.0 | **Objective:** 4.3 | **Verb:** Given a scenario

</details>

---

**Q22.** A DBA discovers that an application's service account has `sysadmin` privileges on the database server. The application only needs to read and write to two specific tables. What should the DBA do?

A. Leave the permissions as-is since the application is working
B. Remove the service account entirely
C. Create a new service account with only `SELECT`, `INSERT`, `UPDATE`, and `DELETE` on the two tables
D. Add MFA to the existing service account

<details><summary>Answer</summary>

**C.** Apply least privilege - create a scoped account with only the permissions the application actually needs. A leaves a massive security gap. B breaks the application. D doesn't reduce the excessive permissions.

**Domain:** 4.0 | **Objective:** 4.3 | **Verb:** Given a scenario

</details>

---

## Explain / Identify

**Q23.** Which ACID property ensures that once a transaction is committed, the data survives even if the server crashes immediately afterward?

A. Atomicity
B. Consistency
C. Isolation
D. Durability

<details><summary>Answer</summary>

**D.** Durability guarantees that committed data persists through crashes, achieved via write-ahead logs and checkpoints. A ensures all-or-nothing execution. B ensures valid state transitions. C prevents interference between concurrent transactions.

**Domain:** 1.0 | **Objective:** 1.1 | **Verb:** Explain

</details>

---

**Q24.** An ORM is generating one SQL query for each related object instead of a single JOIN, causing hundreds of queries for a single page load. What is this problem called?

A. Over-fetching
B. Implicit transactions
C. N+1 query problem
D. Cartesian product

<details><summary>Answer</summary>

**C.** The N+1 problem occurs when the ORM issues 1 query for the parent plus N queries for each related child object. A is retrieving too many columns (`SELECT *`). B is about ORM holding transactions open too long. D is an unintended cross-join.

**Domain:** 1.0 | **Objective:** 1.3 | **Verb:** Explain

</details>

---

**Q25.** During requirements gathering, a gap analysis reveals the current server lacks sufficient IOPS for the projected workload. What does a gap analysis identify?

A. The difference between current assets and what is needed
B. The number of concurrent database users
C. The logical schema design
D. The cardinality between entities

<details><summary>Answer</summary>

**A.** A gap analysis inventories current assets and identifies what's missing versus what's required - in this case, storage with higher IOPS. B is a requirements-gathering input, not a gap analysis output. C and D relate to schema design, not infrastructure assessment.

**Domain:** 1.0 / 2.0 | **Objective:** 1.4 / 2.1 | **Verb:** Explain

</details>

---

**Q26.** Which deployment phase involves loading initial data from CSV files into a newly created database?

A. Installation and configuration
B. Provisioning
C. Modifying
D. Importing

<details><summary>Answer</summary>

**D.** Importing is the phase where initial data is loaded from CSVs, flat files, or another database. A covers DBMS binary installation. B allocates storage and creates schemas. C alters existing configuration.

**Domain:** 2.0 | **Objective:** 2.2 | **Verb:** Explain

</details>

---

**Q27.** A production database server should typically use which type of IP addressing, and why?

A. Dynamic (DHCP) - easier to manage across environments
B. Static - predictable for firewall rules and DNS configuration
C. Dynamic (DHCP) - reduces the risk of IP conflicts
D. Static - required by all DBMS vendors

<details><summary>Answer</summary>

**B.** Production DB servers use static IPs so firewall rules and DNS entries remain stable and predictable. A and C describe DHCP, which is acceptable for dev/test but not production. D is incorrect - it's a best practice, not a vendor requirement.

**Domain:** 2.0 | **Objective:** 2.2 | **Verb:** Explain

</details>

---

**Q28.** A DBA notices CPU usage is at 80%. Is this a problem?

A. Yes - CPU should never exceed 50%
B. No - 80% is always within acceptable range
C. It depends on the baseline - compare against the established normal metrics
D. Yes - the DBA should immediately add more CPUs

<details><summary>Answer</summary>

**C.** Without a baseline, you can't determine if 80% is abnormal. If the baseline is 40%, it's a problem. If it's 78%, it's within normal range. A and D overreact without context. B makes an assumption without evidence.

**Domain:** 3.0 | **Objective:** 3.1 | **Verb:** Explain

</details>

---

**Q29.** Two transactions each hold a lock that the other needs, and neither can proceed. The DBMS detects the situation and terminates one transaction. What is this situation called?

A. Table locking
B. Deadlock
C. Connection pooling exhaustion
D. Replication lag

<details><summary>Answer</summary>

**B.** A deadlock occurs when two or more transactions form a circular wait on locks. The DBMS's deadlock detector terminates one transaction (the "victim") to break the cycle. A is a locking technique, not the circular-wait problem. C is about running out of available connections. D is delay in replication.

**Domain:** 3.0 | **Objective:** 3.1 | **Verb:** Explain

</details>

---

**Q30.** Why should a DBA monitor transaction log file size?

A. Transaction logs only matter during backups
B. An unmanaged transaction log can fill the disk and cause writes to fail
C. Transaction logs are automatically truncated by the OS
D. Transaction log size has no impact on performance

<details><summary>Answer</summary>

**B.** Transaction logs record every data modification. In full recovery model, logs grow continuously until backed up. An unmanaged log can fill the disk, blocking all writes. A is incorrect - logs are critical for ongoing recovery, not just backups. C is false - the DBMS manages logs, not the OS. D is wrong - large logs consume disk space and affect I/O.

**Domain:** 3.0 | **Objective:** 3.1 | **Verb:** Explain

</details>

---

**Q31.** A security patch is released for the DBMS. What should the DBA do BEFORE applying it to production?

A. Apply it immediately to production to minimize exposure
B. Wait for the next scheduled maintenance window without testing
C. Test the patch in a non-production environment first
D. Only apply security patches annually

<details><summary>Answer</summary>

**C.** Security patches should be prioritized but must be tested in a non-production environment before being applied to production to avoid breaking changes. A skips validation. B delays without testing. D creates unacceptable exposure windows.

**Domain:** 3.0 | **Objective:** 3.2 | **Verb:** Explain

</details>

---

**Q32.** What is the correct order for change management?

A. Implement → Approve → Communicate → Validate
B. Plan → Approve → Communicate → Implement → Validate → Document
C. Communicate → Plan → Implement → Approve
D. Implement → Document → Approve → Communicate

<details><summary>Answer</summary>

**B.** The correct change management sequence is: plan the change, get formal approval (CAB), communicate to stakeholders, implement during the approved window, validate the results, and document. All other options have steps out of order - notably, approval must come before implementation.

**Domain:** 3.0 | **Objective:** 3.2 | **Verb:** Explain

</details>

---

**Q33.** An index rebuild is scheduled but must acquire exclusive locks on the table. When should this be performed?

A. During peak business hours for fastest completion
B. Immediately upon detecting any fragmentation
C. During a maintenance window to minimize user impact
D. Only after a full backup has been restored

<details><summary>Answer</summary>

**C.** Integrity checks and index rebuilds that require exclusive locks should be scheduled during maintenance windows when user impact is lowest. A maximizes disruption. B doesn't account for user impact. D is unrelated - backups don't need to be restored before an index rebuild.

**Domain:** 3.0 | **Objective:** 3.2 | **Verb:** Explain

</details>

---

**Q34.** Which encryption method protects data while it travels between the application server and the database server?

A. Transparent Data Encryption (TDE)
B. Column-level encryption
C. In-transit encryption (TLS/SSL)
D. Full-disk encryption (BitLocker)

<details><summary>Answer</summary>

**C.** In-transit encryption (TLS/SSL) protects data moving across the network between client and server. A encrypts data at rest on disk. B encrypts specific columns at rest. D encrypts the entire disk volume at rest.

**Domain:** 4.0 | **Objective:** 4.1 | **Verb:** Explain

</details>

---

**Q35.** A DBA needs to provide realistic test data to the development team without exposing production PII. The production data must remain unchanged. Which technique should they use?

A. Dynamic masking
B. Static masking
C. Cryptographic erasure
D. Full-disk encryption

<details><summary>Answer</summary>

**B.** Static masking permanently alters data in a non-production copy, providing realistic but fictitious data for testing. A masks data at query time in production based on user roles - it doesn't create a sanitized copy. C destroys data by deleting the encryption key. D protects the volume, not individual data values.

**Domain:** 4.0 | **Objective:** 4.1 | **Verb:** Explain

</details>

---

**Q36.** A database stores customer Social Security numbers, medical diagnoses, and credit card numbers. Match each data type to its primary classification or standard:

Which classification applies specifically to medical diagnoses?

A. PII
B. PHI
C. PCI DSS
D. GDPR

<details><summary>Answer</summary>

**B.** Medical diagnoses are Personal Health Information (PHI), governed by HIPAA. A (PII) covers general identifiers like SSN and name. C (PCI DSS) applies to payment card data. D (GDPR) is a regulation about EU residents' personal data, not a data classification category.

**Domain:** 4.0 | **Objective:** 4.2 | **Verb:** Explain

</details>

---

**Q37.** An EU resident requests that a company delete all of their personal data. Which regulation grants this right?

A. PCI DSS
B. HIPAA
C. GDPR
D. SOC 2

<details><summary>Answer</summary>

**C.** GDPR (General Data Protection Regulation) establishes the "right to erasure" for EU/EEA residents. A is a payment card security standard. B is a US healthcare regulation. D is a security audit framework.

**Domain:** 4.0 | **Objective:** 4.2 | **Verb:** Explain

</details>

---

**Q38.** Where should a database server be placed in the network architecture?

A. In the DMZ alongside web servers
B. On an internal network segment, accessible only from the application tier through a firewall
C. Directly accessible from the internet for maximum availability
D. On the same network segment as end-user workstations

<details><summary>Answer</summary>

**B.** The database should never reside in the DMZ or be internet-facing. It belongs on an internal segment, reachable only from authorized application servers through firewall rules. A exposes the database to external threats. C and D violate basic network segmentation principles.

**Domain:** 4.0 | **Objective:** 4.4 | **Verb:** Explain

</details>

---

**Q39.** What is the primary purpose of RPO (Recovery Point Objective)?

A. Define the maximum acceptable downtime
B. Define the maximum acceptable amount of data loss measured in time
C. Determine which DR technique to use
D. Set the backup retention period

<details><summary>Answer</summary>

**B.** RPO defines how much data loss is acceptable, measured in time (e.g., "we can lose at most 1 hour of data"). A describes RTO (Recovery Time Objective). C is a decision that RPO informs, not the definition of RPO itself. D is about retention policy, not recovery objectives.

**Domain:** 5.0 | **Objective:** 5.1 | **Verb:** Explain

</details>

---

**Q40.** A company requires near-zero data loss but can tolerate a few seconds of downtime during failover. Which DR technique is the BEST fit?

A. Daily full backups
B. Log shipping every 4 hours
C. Synchronous database mirroring
D. Asynchronous replication

<details><summary>Answer</summary>

**C.** Synchronous mirroring provides near-zero RPO (every write is confirmed on the mirror before acknowledged) and RTO of seconds for automatic failover. A has RPO of up to 24 hours. B has RPO of up to 4 hours. D risks losing recent transactions because the primary doesn't wait for acknowledgment.

**Domain:** 5.0 | **Objective:** 5.1 | **Verb:** Explain

</details>

---

**Q41.** What is the key difference between a differential backup and an incremental backup?

A. Differential captures changes since the last backup of any type; incremental captures changes since the last full backup
B. Differential captures changes since the last full backup; incremental captures changes since the last backup of any type
C. Both capture the same data but use different compression
D. Differential is faster to restore; incremental is faster to create

<details><summary>Answer</summary>

**B.** Differential = changes since last **full** backup. Incremental = changes since last backup of **any type** (full or incremental). A reverses the definitions. C is incorrect - they capture different data sets. D is partially true (differential is faster to restore) but doesn't describe the fundamental difference.

**Domain:** 5.0 | **Objective:** 5.2 | **Verb:** Explain

</details>

---

**Q42.** A DBA needs to restore a database. They have a full backup from Sunday, and incremental backups from Monday, Tuesday, and Wednesday. The failure occurred Wednesday evening. What must be restored?

A. Sunday full + Wednesday incremental only
B. Sunday full + Monday + Tuesday + Wednesday incrementals in order
C. Wednesday incremental only
D. Sunday full + most recent differential

<details><summary>Answer</summary>

**B.** Incremental backups only contain changes since the last backup, so every incremental must be applied in sequence after the full backup. A would work for a differential (not incremental). C is incomplete - the incremental alone doesn't have all the data. D assumes a differential exists, but the scenario specifies incrementals.

**Domain:** 5.0 | **Objective:** 5.2 | **Verb:** Explain

</details>

---

**Q43.** Before restoring a backup, the DBA compares the backup file's SHA-256 hash against the hash recorded at creation time. The hashes don't match. What should the DBA do?

A. Restore the backup anyway since hash mismatches are common
B. Do not restore - the mismatch indicates corruption or tampering
C. Re-run the hash algorithm to get a third value
D. Restore the backup to a test environment to check for visible errors

<details><summary>Answer</summary>

**B.** A hash mismatch means the backup file has changed since creation - either through corruption or tampering. The DBA should not restore from this file and should use an alternate backup. A ignores a critical integrity check. C doesn't resolve the mismatch. D risks restoring corrupted data.

**Domain:** 5.0 | **Objective:** 5.2 | **Verb:** Explain

</details>

---

**Q44.** A company must retain financial records for 7 years to comply with regulations. Which retention strategy should the DBA implement?

A. Purge backups after 30 days
B. Archive to long-term, low-cost storage with encryption
C. Keep all backups on-site indefinitely
D. Delete all backups after the fiscal year ends

<details><summary>Answer</summary>

**B.** Archive moves aged backups to long-term storage (tape, cold storage, Glacier) for regulatory compliance. Encryption and key retention for the same duration are essential. A and D violate the 7-year retention requirement. C wastes expensive primary storage when cheaper archive options exist.

**Domain:** 5.0 | **Objective:** 5.2 | **Verb:** Explain

</details>

---

## Describe / Recognize

**Q45.** A web application's login form is exploited by an attacker who enters `' OR 1=1 --` as the username. The database returns all user records. Which attack is this?

A. Brute-force attack
B. Denial of Service (DoS)
C. SQL injection
D. On-path attack

<details><summary>Answer</summary>

**C.** SQL injection inserts malicious SQL through input fields that the database executes. The `' OR 1=1 --` pattern bypasses authentication by making the WHERE clause always true. A uses automated password guessing. B overwhelms the server with traffic. D intercepts network communication.

**Best prevention:** Parameterized queries / prepared statements.

**Domain:** 4.0 | **Objective:** 4.5 | **Verb:** Describe

</details>

---

**Q46.** An attacker intercepts the unencrypted connection between a web server and a database server, capturing login credentials and query results. Which type of attack is this?

A. Phishing
B. SQL injection
C. On-path (man-in-the-middle) attack
D. Brute-force attack

<details><summary>Answer</summary>

**C.** An on-path (man-in-the-middle) attack intercepts communication between two systems. The attacker can capture credentials, data, and session tokens. A tricks users through fake emails/websites. B injects malicious SQL through application inputs. D guesses passwords through repeated login attempts.

**Best prevention:** TLS/SSL encryption for all database connections.

**Domain:** 4.0 | **Objective:** 4.5 | **Verb:** Describe

</details>

---

**Q47.** A DBA arrives at work to find all database files encrypted with a message demanding payment in cryptocurrency. Backups stored on the same server are also encrypted. Which attack occurred, and what is the critical lesson?

A. DoS attack - implement rate limiting
B. SQL injection - use parameterized queries
C. Ransomware - maintain offline/immutable backups separate from the production server
D. Brute-force attack - enforce account lockout policies

<details><summary>Answer</summary>

**C.** Ransomware (a type of malware) encrypts files and demands payment. The critical lesson is that backups stored on the same server are vulnerable to the same attack. Offline or immutable backups stored separately are essential for recovery. A, B, and D describe different attacks with different mitigations.

**Domain:** 4.0 | **Objective:** 4.5 | **Verb:** Describe

</details>
