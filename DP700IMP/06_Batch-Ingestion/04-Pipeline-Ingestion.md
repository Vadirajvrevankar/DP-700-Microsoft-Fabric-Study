# DP-700 — Data Copy Options: Explained Notes

## 1. Copy Activity vs Copy Job

### Copy activity
- Runs inside a **Data pipeline**.
- Copies data from a source to a destination (sink).
- Offers detailed control over source/sink settings, staging, partitioning, parallelism, and fault tolerance.
- Useful when detailed pipeline control is required.

**Example:** Copy data from Azure SQL Database into a Fabric Lakehouse using a pipeline.

### Copy job
- A simplified, pipeline-free copy experience.
- Designed for **incremental and CDC-driven copy** scenarios.
- Reduces the need to build and maintain custom watermark tables and watermark logic.
- Supports native CDC-based incremental copy, including deletes, according to the supplied study notes.
- SCD Type 2 and auto-partitioning are listed as **Preview** features in the supplied notes.

**Exam memory trick:** Incremental/CDC copy → Copy job. Detailed pipeline control → Copy activity.

## 2. Incremental Copy and CDC

### Incremental copy
- Copies new or changed data rather than reloading the entire dataset.
- A watermark can track the last processed point, such as a timestamp or increasing ID.
- Hand-built watermark solutions require maintaining the tracking logic.

### Change Data Capture (CDC)
- Captures changes made to source data.
- Changes can include inserts, updates, and deletes, depending on source and connector support.
- Copy job's native CDC-based incremental copy is designed to reduce custom tracking work.

**Exam clue:** Incremental or CDC-driven copy with less custom watermark logic → consider Copy job.

## 3. Fault Tolerance: Skip Incompatible Rows, Logging, and Consistency

### Skip incompatible rows
- Allows a copy to continue when certain rows are incompatible with the destination or expected schema.
- This can prevent a run from failing because of problematic rows.
- But skipped rows mean some source data may not reach the destination.

### Enable logging
- Logging records information about skipped or problematic data so it can be investigated.
- Pair **Skip incompatible rows** with **Enable logging**.

### Data consistency verification
- Use it when data completeness and consistency matter.
- It helps verify that the copy result is consistent with the source according to the feature's checks.

**Important:** Fault tolerance alone can hide data loss. A successful run does not necessarily mean every source row was copied.

**Exam memory trick:** Skip rows → Logging → Consistency verification when completeness matters.

## 4. Partition Option and Degree of Copy Parallelism

These are two separate settings that work together.

### Partition option
- Divides source data into partitions so it can be read in parallel.

### Degree of copy parallelism
- Controls how many copy operations can run in parallel.

### How they work together
- Enable a **Partition option** before increasing **Degree of copy parallelism**.
- Parallelism without partitioning is a no-op in the scenario described by the supplied notes.
- Partitioning without enough parallelism—or parallelism without partitioning—may under-deliver.

**Exam memory trick:** Partition the data first, then tune parallelism.

## 5. Staging: External vs Workspace

- Staging is an intermediate location used during certain copy operations.
- For a Copy activity expected to run longer than **60 minutes**, use **external staging**, not workspace staging, according to the supplied study notes.

**Exam clue:** Copy activity expected to exceed 60 minutes → external staging.

## 6. `COPY INTO`

`COPY INTO` is a code-first bulk-loading option for a Fabric Warehouse table.

Choose it when all these conditions match:
1. The destination is a **Warehouse table**.
2. The source is already in **external Azure storage**.
3. The team is **T-SQL-first**.
4. No transformation is required during the load.

### Key limitations and clues
- Warehouse-only in this comparison.
- Source must be external Azure storage.
- Performs **zero transformation**.
- It can be a simpler fit than a pipeline for this specific scenario.

**Exam memory trick:** Warehouse + Azure storage + T-SQL team + no transformation → `COPY INTO`.

## 7. Upsert Write Behavior

- **Upsert** is a sink-side write behavior: it determines how records are written to the destination.
- The supplied notes list database-family connectors such as **Azure SQL Database, Warehouse, Lakehouse tables, and Dataverse**.
- File-based sinks do **not** have the Upsert write behavior described in these notes.

**Exam clue:** Upsert is a destination/sink setting, not a source setting.

## 8. Binary Copy vs Tabular Copy

| Type | Explanation |
|---|---|
| **Binary copy** | Byte-for-byte copying; no type awareness |
| **Tabular copy** | Schema-aware conversion through an interim type system |

**Memory aid:** Binary = bytes as-is. Tabular = schema-aware conversion.

## 9. Connector Coverage

The supplied study notes give these approximate counts:
- Overall ecosystem: **170+ sources**.
- Copy activity / Copy job: approximately **50+ / 40+** sources.
- Dataflow Gen2: approximately **150+** sources.
- Connector coverage differs; the tools do not have identical source lists.

These counts can change. Verify current connector support when a specific source matters.

## 10. Choosing the Right Tool

| Requirement | Best-fit option |
|---|---|
| Incremental/CDC copy with less custom watermark work | **Copy job** |
| Detailed source/sink settings, staging, partitioning, and fault tolerance | **Copy activity** |
| Copy activity expected to run longer than 60 minutes | **External staging** |
| Warehouse destination + external Azure storage + T-SQL-first + no transformation | **`COPY INTO`** |
| Power Query-based, transformation-rich data integration | **Dataflow Gen2** |
| Orchestrate multiple activities/workflows | **Pipeline** |
| Byte-for-byte copy | **Binary copy** |
| Schema-aware conversion | **Tabular copy** |

## 11. Exam Tips

- Prefer **Copy job** for incremental/CDC-driven copy when it avoids hand-built watermark logic.
- Pair **Skip incompatible rows** with **Enable logging** and, when completeness matters, **Data consistency verification**.
- Enable a **Partition option** before increasing **Degree of copy parallelism**.
- Use **external staging** for Copy activities expected to run longer than **60 minutes**.
- Use **`COPY INTO`** for a Warehouse table loaded from external Azure storage by a T-SQL-first team when no transformation is needed.
- **Upsert** is a sink-side setting on database-family connectors listed in the study notes; file-based sinks do not have it.
- Copy job headline additions in the supplied notes: native CDC-based incremental copy (including deletes), SCD Type 2 (Preview), auto-partitioning (Preview), and no pipeline canvas.
- Binary copy is byte-for-byte; tabular copy is schema-aware.

## 12. Key Takeaways

- Copy activity gives detailed pipeline control over source/sink settings, staging, partitioned parallel reads, and fault-tolerant row/file skipping.
- Copy job simplifies incremental and CDC-based copy and can replace custom watermark logic in suitable scenarios.
- `COPY INTO` is a focused, code-first bulk-load path for Warehouse tables from external Azure storage.
- Dataflow Gen2 is the Power Query-based, transformation-rich option; pipelines handle orchestration and broader workflows.
- Fault tolerance can prevent hard failures, but skipped rows/files need logging and consistency verification to avoid silent data loss.

## 13. One-Minute Revision

- **CDC/incremental?** Copy job.
- **Detailed pipeline controls?** Copy activity.
- **Skip incompatible rows?** Enable logging; add consistency verification when completeness matters.
- **Parallel copy?** Partition option + degree of parallelism.
- **Longer than 60 minutes?** External staging.
- **Warehouse + Azure storage + T-SQL + no transformation?** `COPY INTO`.
- **Upsert?** Sink-side behavior for supported database-family connectors.
- **Bytes as-is?** Binary copy. **Schema-aware?** Tabular copy.
