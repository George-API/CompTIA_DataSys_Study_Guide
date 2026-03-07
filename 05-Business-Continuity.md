# 5.0 Business Continuity — 12%

Lowest-weighted domain — be concise and focus on key distinctions (full vs. incremental vs. differential, RPO vs. RTO, replication vs. mirroring). No filler.

---

## 5.1 Explain the Importance of Disaster Recovery and Relevant Techniques

### Disaster Recovery (DR) Planning

DR planning ensures the organization can restore database services after a catastrophic event (hardware failure, natural disaster, cyberattack). A DR plan defines **who** does **what**, **when**, and with **what resources**.

### DR Documentation

| Document | Purpose |
|----------|---------|
| **Manuals** | Step-by-step recovery procedures for each critical system |
| **System security plan** | Documents security controls, risk assessments, and contingency measures for the database environment |
| **Continuity of operations plan (COOP)** | Broader organizational plan for maintaining essential functions during and after a disaster |
| **Build documentation** | Instructions to rebuild the database server from scratch — OS install, DBMS configuration, schema scripts, data restore |

### DR Techniques

| Technique | How It Works | RPO / RTO Impact |
|-----------|-------------|-----------------|
| **Replication** | Continuously copies data changes to one or more standby servers (synchronous or asynchronous) | Synchronous: near-zero RPO, fast RTO. Asynchronous: seconds-to-minutes RPO, fast RTO |
| **Log shipping** | Periodically ships transaction log backups to a standby server and restores them | RPO = interval between log shipments (minutes to hours). RTO = time to bring standby online |
| **High availability (HA)** | Clustering, failover groups, or always-on configurations that automatically switch to a standby node on failure | Near-zero RPO and RTO; designed for minimal downtime |
| **Mirroring** | Real-time copy of the database to a mirror server; can be synchronous (high safety) or asynchronous (high performance) | Synchronous: near-zero RPO. RTO = seconds for automatic failover |

**Exam tip:** Know the trade-off — synchronous replication/mirroring provides the lowest RPO but adds latency to every write because the primary waits for the secondary to acknowledge. Asynchronous is faster but risks losing the most recent transactions.

### DR Plan Testing

| Metric | Definition | Key Question |
|--------|------------|--------------|
| **Recovery Point Objective (RPO)** | Maximum acceptable amount of data loss measured in time | "How much data can we afford to lose?" |
| **Recovery Time Objective (RTO)** | Maximum acceptable downtime before services must be restored | "How quickly must we be back online?" |

- **RPO drives backup frequency.** If RPO = 1 hour, you need backups (or log shipping) at least every hour.
- **RTO drives architecture.** If RTO = minutes, you need HA clustering or hot standby — cold restores from tape won't meet the target.

Test types:

- **Tabletop exercise** — walk through the plan on paper; low cost, catches procedural gaps.
- **Simulation** — simulate a failure in a test environment; validates technical steps.
- **Full failover test** — actually fail over to the DR site; highest confidence but highest risk and cost.

### Transition / Failback to Normal Operations

After a DR event, returning to the primary site requires:

1. Verify the primary environment is fully restored and healthy.
2. Resynchronize data from the DR site back to primary.
3. Validate data integrity (row counts, checksums, referential integrity).
4. Switch traffic back to primary (DNS change, connection string update, load-balancer reconfiguration).
5. Monitor closely for a defined period to confirm stability.
6. Update DR documentation with lessons learned.

### Risk Assessment and Continuity Testing

- **Risk assessment** — identify threats (hardware failure, ransomware, natural disasters), their likelihood, and their impact on database availability.
- **Continuity testing** — regularly test that backup, replication, and failover mechanisms actually work under realistic conditions.
- Document findings and remediate gaps before the next test cycle.

**What they might ask:** "The DR plan specifies an RPO of 15 minutes but backups run only once per day — what's the problem?" → The backup frequency does not meet the RPO. Implement transaction log backups or replication at 15-minute intervals.

---

## 5.2 Explain Backup and Restore Best Practices and Processes

### Backup Types

| Type | What It Captures | Restore Speed | Storage Cost | When to Use |
|------|-----------------|---------------|--------------|-------------|
| **Full backup** | Entire database — every page of data | Fastest restore (single file) | Highest (largest file) | Weekly or as the baseline for differential/incremental chains |
| **Differential** | All changes since the **last full backup** | Moderate — restore full + latest differential | Grows over time between fulls | Daily; balances restore speed and storage |
| **Incremental** | Only changes since the **last backup of any type** | Slowest — restore full + every incremental in sequence | Smallest individual files | When storage is limited and backup windows are tight |

**Critical distinction (exam favorite):**

- **Differential** = changes since last **full**.
- **Incremental** = changes since last **backup** (full or incremental).
- Restoring from incrementals requires the full backup plus **every** incremental in order. Restoring from a differential requires only the full backup plus the **most recent** differential.

### Database Dumping

- A logical export of database objects and data (e.g., `pg_dump`, `mysqldump`, `expdp`).
- Produces a portable script or file that can recreate the database on the same or different platform.
- Slower than physical backups for large databases but useful for migrations and selective restores.

### Schedule and Automate Backups

- Use the DBMS's built-in scheduler (SQL Server Agent, MySQL Event Scheduler, `cron` + `pg_dump`) or an enterprise backup tool.
- Automate the full chain: full → differential/incremental → log backups.
- Align schedule with RPO — shorter RPO requires more frequent backups.

### Test Backups

- A backup that has never been restored is a **hope**, not a backup.
- Periodically restore backups to a test environment and verify data integrity.
- Document the restore time to validate that it meets the RTO.

### Validate Backup Hash

- Generate and store a hash (SHA-256, MD5) of backup files at creation time.
- Before restoring, compare the current hash to the stored value.
- A mismatch indicates corruption or tampering — do not restore from that file.

### Storage Location

| Location | Pros | Cons |
|----------|------|------|
| **On-site** | Fast restore (local network speed); immediate access | Vulnerable to the same disaster that takes out the primary (fire, flood) |
| **Off-site** | Survives local disasters; geographic separation | Slower restore (network transfer); additional cost; encryption in transit required |

**Best practice:** Maintain copies in **both** locations. The 3-2-1 rule — 3 copies, 2 different media types, 1 off-site — is a widely accepted standard.

### Retention Policy

| Strategy | Description | Use Case |
|----------|-------------|----------|
| **Purge** | Delete backups after the retention period expires | Routine operational backups where long-term storage isn't required |
| **Archive** | Move aged backups to long-term, low-cost storage (tape, cold storage, Glacier) | Regulatory or legal hold requirements; historical data preservation |

- Retention periods are driven by business policy and regulatory requirements (e.g., PCI DSS mandates 1 year of audit log retention).
- Ensure archived backups are encrypted and that encryption keys are also retained for the same duration.

**Exam tip:** "A company must retain financial records for 7 years — which retention strategy applies?" → **Archive** to long-term storage with appropriate encryption and key management.
