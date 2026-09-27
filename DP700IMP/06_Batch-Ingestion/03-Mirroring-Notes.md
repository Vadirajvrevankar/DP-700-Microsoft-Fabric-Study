# DP-700 — Microsoft Fabric Mirroring: Complete Explained Notes

## 1. What Is Mirroring?

**Mirroring** makes data from a source system available for analytics in Microsoft Fabric and OneLake without requiring a traditional copy pipeline for supported scenarios.

### Example
A company has a production database containing customer orders.

- With a traditional pipeline, the team builds and manages a copy process to move data into an analytics destination.
- With mirroring, Fabric continuously replicates changes from a supported source and makes the mirrored data available for analytics.

**Remember:** Mirroring is for analytics availability. It does not replace the original operational source.

## 2. Three Types of Mirroring

### A. Database Mirroring — Full Replication
- Replicates a supported source database into OneLake as Delta tables.
- Designed for continuous replication and analytics availability.
- Changes can publish as fast as every **15 seconds**—near-real-time, not instantaneous.
- Mirrored data in Fabric is **read-only**; writes are made to the original source system.

**Use when:** A fully supported operational database needs continuous, pipeline-free analytics availability.

**Exam clue:** Full database replication.

### B. Metadata Mirroring — Catalog Metadata and Shortcuts
- Synchronizes catalog metadata.
- Uses shortcuts to reference existing data rather than copying the underlying data.
- The supplied study material identifies **Unity Catalog and Dremio** as metadata-mirroring sources.

**Use when:** Working with a catalog-based source and metadata synchronization is sufficient.

**Exam clue:** Catalog structure is synchronized; the underlying data is not duplicated.

### C. Open Mirroring — Application-Written Change Data
- Lets an application write change data into a landing zone using an open, public specification.
- Supports application-driven publishing of changes into Fabric.

**Use when:** An application needs to publish change data using the open mirroring specification.

**Exam clue:** An application writes changes to a landing zone.

## 3. When to Choose Mirroring

Default to database mirroring when:
- The operational source is fully supported.
- You need continuous analytics availability.
- You want to avoid building and maintaining a traditional copy pipeline.

Choose metadata mirroring for catalog-based sources such as Unity Catalog when the requirement is to synchronize metadata and reference existing data without duplicating it.

Use open mirroring when an application writes change data into a landing zone according to the open specification.

## 4. Storage Allowance and Compute Costs

The supplied study material states:
- Free storage allowance: **1 TB of OneLake storage per purchased capacity unit (CU)**.
- Replication/background compute is **free**.
- Query compute is **billed normally**.

### Why monitor storage?
Mirrored data consumes OneLake storage. Monitor usage against the free allowance so that exceeding it does not unexpectedly lead to standard OneLake storage billing.

**Exam distinction:** Free replication compute does not mean query compute is free.

## 5. Database Mirroring Latency

- Database mirroring changes can publish as fast as every **15 seconds**.
- This is described as **near-real-time**, not instantaneous.
- Do not interpret “near-real-time” as zero latency.

**Exam clue:** If a question asks whether mirroring is instantaneous, the answer is no; changes can publish as fast as 15 seconds according to the supplied notes.

## 6. Read-Only Behavior

- Mirrored data in Fabric is **read-only**.
- To modify the source data, write to the original source system.
- Mirroring makes data available for analytics; it is not the write interface for the operational source.

**Memory aid:** Read in Fabric; write to the source.

## 7. Delta Retention and Time Travel

Adjust Delta retention deliberately for a mirrored database.

| Retention choice | Effect |
|---|---|
| Shorter retention | Keeps less historical data and can save storage |
| Longer retention | Preserves more time-travel history and can use more storage |

Choose a retention period based on how much history the organization needs and its storage requirements.

## 8. Source Support Changes

The supported source list for database mirroring grows over time, and source availability or preview status can change.

The supplied study material mentions Oracle, SAP, BigQuery, and MySQL as recent additions. Treat this as a time-sensitive study note, not a permanent list.

**Exam preparation:** Verify the current supported-source list and preview labels close to exam day.

## 9. Mirroring vs Shortcut vs Pipeline Copy

| Option | Best fit |
|---|---|
| **Database mirroring** | Continuous replication of a supported operational source |
| **Metadata mirroring** | Synchronize catalog metadata and reference data with shortcuts, avoiding data duplication |
| **Open mirroring** | An application writes change data to a landing zone using an open specification |
| **Shortcut** | Reference already-queryable data without copying it |
| **Pipeline copy** | Copy data when transformation, format conversion, or a separate snapshot is needed |

**Decision shortcut:**
- Continuous replication → Mirroring.
- Reference compatible existing data → Shortcut.
- Transform, convert, or create a decoupled snapshot → Pipeline copy.

## 10. Exam Tips

- **Three flavors:** database mirroring (full replication), metadata mirroring (shortcuts; Unity Catalog/Dremio in these notes), and open mirroring (application writes to a landing zone).
- **Free storage:** 1 TB of OneLake storage per purchased CU, according to the supplied study material.
- **Compute:** Replication/background compute is free; query compute is billed normally.
- **Latency:** Database mirroring changes can publish as fast as 15 seconds—near-real-time, not instantaneous.
- **Read-only:** Mirrored data in Fabric is read-only; writes go to the original source.
- **Metadata mirroring:** Synchronizes catalog structure and uses shortcuts, avoiding data movement.
- **Source list:** Check current support and preview labels close to exam day.

## 11. Key Takeaways

- Database mirroring replicates entire supported databases into OneLake Delta tables.
- Metadata mirroring (Unity Catalog and Dremio in these notes) synchronizes catalog structure and uses shortcuts, so the underlying data is not duplicated.
- Open mirroring lets applications write change data to a landing zone according to an open specification.
- Mirroring storage is free up to the stated 1 TB per purchased capacity unit; background replication compute is free, while query compute is billed normally.
- Choose mirroring for continuous replication, shortcuts for referencing already-queryable data, and pipeline copy for scenarios requiring transformation or a decoupled snapshot.

## 12. Quick Revision

| Question clue | Answer |
|---|---|
| Full database replication | Database mirroring |
| Catalog metadata + shortcuts, no data duplication | Metadata mirroring |
| App writes change data to landing zone | Open mirroring |
| Free storage allowance | 1 TB per purchased CU (per supplied notes) |
| Replication/background compute | Free |
| Query compute | Billed normally |
| Fastest stated publish interval | 15 seconds; near-real-time |
| Can you write to mirrored data in Fabric? | No; write to original source |
| Shorter Delta retention | Less history; can save storage |
| Longer Delta retention | More time-travel history; may use more storage |
| Continuous supported-source analytics | Mirroring |
| Reference existing query-compatible data | Shortcut |
| Transform or convert data | Pipeline copy |
