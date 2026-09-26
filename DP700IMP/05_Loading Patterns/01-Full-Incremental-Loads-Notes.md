# DP-700 — Full & Incremental Load Patterns

## 1. Overview
This topic covers full vs. incremental loads, watermarks, retry-safe loading, Delta partition replacement, and change detection.

**Memory:** Full load = reload everything | Incremental = load changes | Watermark = last processed point | MERGE = retry-safe upsert | replaceWhere = targeted partition replacement.

## 2. Full Load
A full load reloads the complete source dataset.

**Advantages:** simple, self-correcting, does not require a change-detection signal.  
**Disadvantages:** can be expensive and slow at scale.

**Use when:** source is small, low-change, or difficult to track incrementally.

> **Exam tip:** Full load = simple and self-correcting, but expensive at scale.

## 3. Incremental Load
An incremental load processes only new or changed data since the prior successful run.

**Advantages:** lower cost per run and useful for large, frequently refreshed sources.  
**Disadvantages:** correctness depends on the reliability of the change-detection signal.

**Use when:** source is large, freshness matters, and a reliable change signal exists.

| Full load | Incremental load |
|---|---|
| Entire dataset | Only new/changed data |
| Simple | More complex |
| Can be expensive at scale | Lower cost per run |
| No change signal needed | Requires reliable change signal |

## 4. Watermark
A watermark records the point up to which data has been processed. A later run uses it to identify records to extract.

Example concept:
```sql
WHERE ModifiedDate > @Watermark
```

### Durable storage
Store the authoritative watermark in a **durable control table**, not only in a pipeline variable or notebook-local variable that resets between runs.

Example:

| PipelineName | WatermarkValue |
|---|---|
| CustomerLoad | 2026-09-20 00:00:00 |
| OrdersLoad | 2026-09-21 00:00:00 |

## 5. Correct Watermark Sequence
Always follow:

1. Read old watermark.
2. Extract changes past it.
3. Load the data.
4. Commit successfully.
5. Advance the watermark **last**.

**Memory:** READ → EXTRACT → LOAD → COMMIT → ADVANCE.

### Why last?
If the watermark advances before the data load succeeds, a failed run can cause the next run to skip records that were never loaded.

### Transaction rule
- **T-SQL:** update the watermark as the last step of the same transaction as the load.
- **Delta:** advance the watermark immediately after a successful `MERGE INTO` commit.

## 6. Idempotency
A load is idempotent when repeating the same operation does not create unintended duplicate or inconsistent results.

This matters for retries, manual reruns, and uncertain run outcomes.

> **IDEMPOTENT = SAFE TO REPEAT**

## 7. MERGE / MERGE INTO
`MERGE` (T-SQL) and `MERGE INTO` (Delta) implement business-key-based upserts:

- Matched key → update
- Unmatched key → insert

Conceptual T-SQL example:
```sql
MERGE INTO Target AS T
USING Source AS S
ON T.BusinessKey = S.BusinessKey
WHEN MATCHED THEN
    UPDATE SET T.Value = S.Value
WHEN NOT MATCHED THEN
    INSERT (BusinessKey, Value)
    VALUES (S.BusinessKey, S.Value);
```

Conceptual Delta example:
```sql
MERGE INTO target_table AS T
USING source_table AS S
ON T.BusinessKey = S.BusinessKey
WHEN MATCHED THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *;
```

Actual syntax depends on engine and schema.

### Why MERGE?
A plain append/insert does not inherently prevent duplicates after retries. `MERGE` is the default pattern for retry-tolerant incremental upserts.

**Exam clue:** “Duplicate rows after retry” → consider missing `MERGE` / `MERGE INTO`.

## 8. Append vs. MERGE

| Operation | Behavior | Typical use |
|---|---|---|
| `append` | Adds rows; not inherently idempotent | Raw, never-reprocessed Bronze ingestion |
| `INSERT` | Adds rows; not inherently idempotent | Insert-only patterns with separate safeguards |
| `MERGE` | Updates matching keys and inserts new keys | Retry-safe incremental upserts |
| `MERGE INTO` | Delta upsert | Retry-safe Delta incremental loads |

Default to `MERGE` / `MERGE INTO` when a load must tolerate retries. Reserve plain append for raw Bronze ingestion that is never reprocessed.

## 9. Delta overwrite + replaceWhere
`replaceWhere` scopes a Delta overwrite to rows matching a predicate, commonly a partition.

Use it when the source delivers a **complete, self-contained partition per run**. It can be simpler than row-by-row `MERGE INTO` and idempotent when used correctly.

Illustrative example:
```python
(df.write
   .format("delta")
   .mode("overwrite")
   .option("replaceWhere", "OrderDate = '2026-09-20'")
   .saveAsTable("FactOrders"))
```
The predicate must match the intended scope, and the incoming data must be complete for that scope.

### Important distinction
- `overwrite` without a predicate can replace the full target table.
- `overwrite` + `replaceWhere` replaces only rows matching the predicate.

**Memory:** FULL PARTITION PER RUN → `overwrite` + `replaceWhere`.

## 10. Change Detection Options

### Modified-date column
Uses a source column that records when a row was last changed. Extract rows newer than the watermark.

- Good for inserts and updates when the timestamp is reliable.
- Simple to implement.
- **Misses hard deletes**, because deleted rows may no longer exist to query.

### CDC (Change Data Capture)
Captures source changes, including delete events when supported and configured.

- Useful when inserts, updates, and deletes must be tracked.
- Requires more setup than a simple modified-date filter.

### Mirroring
Maintains a copy/feed of supported source data in Fabric. In this study topic, it is an option for capturing deletes, subject to source and feature support.

| Method | Inserts | Updates | Hard deletes |
|---|---|---|---|
| Modified-date column | Yes | Yes, if timestamp is updated reliably | No |
| CDC | Yes | Yes | Yes, when supported/configured |
| Mirroring | Yes | Yes | Can capture deletes for supported sources |

**Exam rule:** If deletes do not matter, a modified-date column may be sufficient. If deletes matter, consider CDC or mirroring.

## 11. Scenario Questions

### Scenario 1
A small reference table changes rarely and has no reliable change signal.  
**Answer: Full load.** Simple and self-correcting.

### Scenario 2
A large source needs frequent refreshes and has a reliable modified-date field; hard deletes do not matter.  
**Answer: Incremental load using a modified-date watermark.**

### Scenario 3
Rows are duplicated after a pipeline retry.  
**Answer: Use `MERGE` / `MERGE INTO` keyed by business key.** Append alone is not idempotent.

### Scenario 4
The source delivers a complete partition each run.  
**Answer: Delta `overwrite` + `replaceWhere`.**

### Scenario 5
Hard deletes must be reflected in the target.  
**Answer: CDC or mirroring**, if supported/configured.

### Scenario 6
A pipeline must save the latest processed timestamp.  
**Answer: Update the durable watermark only after successful load commit.**

## 12. Common Exam Traps
1. **Watermark in a temporary variable:** use a durable control table.
2. **Advance watermark before loading:** wrong; commit first, watermark last.
3. **Append for retry-safe upserts:** append alone can duplicate rows; use `MERGE`.
4. **MERGE for every partition refresh:** a complete partition may be simpler with `overwrite` + `replaceWhere`.
5. **Confusing replaceWhere with full overwrite:** it scopes the overwrite to matching rows.
6. **Modified date captures deletes:** false; hard deletes are missed.
7. **Watermark before Delta merge commit:** wrong; commit first.
8. **Incremental load is automatically correct:** false; it depends on the change signal.

## 13. Best Practices
- Keep the authoritative watermark in a durable control table.
- Read → extract → load → commit → advance watermark.
- Use `MERGE` / `MERGE INTO` for retry-tolerant upserts.
- Use append for raw Bronze data only when it is never reprocessed, per this study pattern.
- Use `overwrite` + `replaceWhere` for complete, self-contained partitions.
- Choose change detection based on whether deletes matter.

## 14. Key Takeaways
- Full load is simple and self-correcting but can be expensive at scale.
- Incremental load is cheaper per run but depends on a reliable change signal.
- Watermarks belong in durable control tables.
- T-SQL watermark update: last step of the same transaction.
- Delta watermark update: after successful `MERGE INTO` commit.
- `MERGE` / `MERGE INTO` are the default idempotent upsert pattern.
- `append` / `INSERT` alone are not idempotent.
- `replaceWhere` scopes Delta overwrite to matching partitions.
- Modified-date columns miss hard deletes; CDC and mirroring can capture them when supported.

## 15. One-Minute Revision

```text
FULL LOAD → Reload all → Simple, can be expensive
INCREMENTAL → Load changes → Needs reliable change signal

WATERMARK:
Read old → Extract → Load → Commit → Advance LAST
Store in durable control table

RETRY-SAFE UPSERT:
MERGE / MERGE INTO
Matched → Update
Not matched → Insert

COMPLETE PARTITION:
overwrite + replaceWhere

CHANGE DETECTION:
Modified date → Inserts/updates; misses hard deletes
CDC / Mirroring → Delete capture, if supported
```

## Final Memory Formula
> **SMALL / LOW CHANGE → FULL LOAD**  
> **LARGE / HIGH FRESHNESS → INCREMENTAL**  
> **WATERMARK → DURABLE CONTROL TABLE**  
> **WATERMARK UPDATE → LAST**  
> **RETRY-SAFE UPSERT → MERGE**  
> **COMPLETE PARTITION → overwrite + replaceWhere**  
> **MODIFIED DATE → NO HARD DELETE CAPTURE**  
> **DELETES MATTER → CDC OR MIRRORING**
