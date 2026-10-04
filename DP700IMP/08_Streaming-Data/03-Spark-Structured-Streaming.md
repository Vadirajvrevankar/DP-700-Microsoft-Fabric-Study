# DP-700 — Spark Structured Streaming Notes

## 1. Overview

Spark Structured Streaming is used to process continuously arriving data using a streaming DataFrame API.

```text
Streaming Source
      ↓
   readStream
      ↓
 Transformations
      ↓
   writeStream
      ↓
Delta Lake / Other Sink
```

The most important concepts are:

- `checkpointLocation`
- `withWatermark()`
- Output modes
- `foreachBatch`
- `MERGE`
- `trigger(processingTime=...)`
- `availableNow=True`
- Native Execution Engine and Structured Streaming

---

## 2. `readStream` and `writeStream`

Batch Spark uses:

```python
spark.read
df.write
```

Structured Streaming uses:

```python
spark.readStream
df.writeStream
```

Example:

```python
df = (
    spark.readStream
    .format("cloudFiles")
    .load("/path/input")
)
```

Then:

```python
query = (
    df.writeStream
    .format("delta")
    .option("checkpointLocation", "/checkpoints/orders")
    .outputMode("append")
    .start("/tables/orders")
)
```

Two important streaming-specific settings are:

1. `checkpointLocation`
2. `outputMode`

---

## 3. `checkpointLocation`

A checkpoint stores information about the progress of a streaming query.

Think:

> **Checkpoint = Where did my streaming pipeline stop?**

It tracks information such as:

- Source offsets
- Committed batch information
- Streaming state metadata

If the application fails, Spark can use the checkpoint to determine where it was and safely continue.

### Memory Trick

> **Checkpoint = Where did I stop?**

---

## 4. One Dedicated Checkpoint per Streaming Query

Each streaming query should have its own checkpoint location.

Avoid:

```text
Query A ──┐
          ├── /checkpoints/common
Query B ──┘
```

Prefer:

```text
Query A → /checkpoints/queryA
Query B → /checkpoints/queryB
```

Each query has its own offsets, progress, state, and batch information.

### Exam Tip

> **One streaming query → one checkpoint location**

---

## 5. `withWatermark()`

Some streaming operations need Spark to remember state.

For example:

```python
dropDuplicates()
```

Spark needs to remember previously seen IDs.

If the stream runs forever, state could grow indefinitely.

This is called **state growth**.

A watermark tells Spark how long it should retain relevant streaming state.

Example:

```python
df = df.withWatermark("event_time", "10 minutes")
```

Conceptually:

```text
Current event time = 12:00
Watermark = 11:50
```

Spark can eventually remove state that is no longer relevant.

### Memory Trick

> **Watermark = control how long streaming state is kept**

---

## 6. Watermark + `dropDuplicates`

A streaming deduplication operation needs state.

Example:

```python
df.dropDuplicates(["EventID"])
```

Better pattern:

```python
df = (
    df.withWatermark("EventTime", "10 minutes")
      .dropDuplicates(["EventID"])
)
```

### Exam Tip

> Pair `dropDuplicates` with `withWatermark` to bound state growth.

---

## 7. Watermark + Windowed Aggregation

Windowed aggregations also maintain state.

Example:

```text
10:00–10:05 → COUNT
10:05–10:10 → COUNT
10:10–10:15 → COUNT
```

Use a watermark to help bound that state:

```python
df.withWatermark("event_time", "10 minutes")
```

### Exam Tip

> Pair windowed aggregations with `withWatermark()` to control state growth.

### Trade-Off

Watermarks bound state growth, but data arriving beyond the configured lateness tolerance may no longer be incorporated into the relevant state/window.

> **More tolerance → more state**

> **Less tolerance → less state, but greater risk of late data being excluded**

---

## 8. Output Modes

Structured Streaming has three important output modes:

```text
append
update
complete
```

The choice depends on whether the query aggregates data and how the sink needs to receive results.

---

## 9. Append Mode

```python
.outputMode("append")
```

Append mode writes newly added/final rows.

```text
Input:
A
B
C

Output:
A
B
C
```

Previously emitted results are not subsequently revised.

### Exam Tip

> `append` = no revisions, no aggregation required.

### Memory Trick

> **Append = New rows**

---

## 10. Update Mode

```python
.outputMode("update")
```

Update mode outputs only rows that changed since the previous trigger.

Example:

```text
Previous:
Device A → 10

New result:
Device A → 12
```

Output:

```text
Device A → 12
```

### Memory Trick

> **Update = Changed rows**

---

## 11. Complete Mode

```python
.outputMode("complete")
```

Complete mode outputs the entire current result table every time.

Example:

```text
Device A → 12
Device B → 20
Device C → 15
```

The complete result is output again.

### Memory Trick

> **Complete = Full result**

---

## 12. Append vs Update vs Complete

| Mode | What gets written? | Key idea |
|---|---|---|
| **Append** | New/final rows | No revisions |
| **Update** | Changed rows | Only changed results |
| **Complete** | Entire result | Full rewrite |

### Quick Exam Recognition

| Question wording | Answer |
|---|---|
| No revisions / no aggregation | **Append** |
| Only changed aggregation results | **Update** |
| Entire aggregation result every time | **Complete** |

---

## 13. `foreachBatch`

`foreachBatch` is important when a streaming pipeline needs an upsert or `MERGE`.

Suppose streaming data needs to:

```text
INSERT new records
UPDATE existing records
```

This is an **upsert**.

The pattern is:

```text
Streaming Data
      ↓
writeStream
      ↓
foreachBatch
      ↓
Micro-batch DataFrame
      ↓
MERGE
      ↓
Delta Table
```

### Exam Memory

> **Streaming + Delta MERGE → `foreachBatch`**

---

## 14. Why Use `foreachBatch` for MERGE?

`foreachBatch` gives each micro-batch to your function as a normal DataFrame.

You can then execute batch-style Delta logic such as `MERGE`.

Conceptually:

```python
def upsert_to_delta(batch_df, batch_id):
    batch_df.createOrReplaceTempView("updates")

    batch_df.sparkSession.sql("""
        MERGE INTO target AS t
        USING updates AS s
        ON t.CustomerID = s.CustomerID

        WHEN MATCHED THEN
            UPDATE SET *

        WHEN NOT MATCHED THEN
            INSERT *
    """)
```

Then:

```python
query = (
    streaming_df.writeStream
    .foreachBatch(upsert_to_delta)
    .option("checkpointLocation", "/checkpoints/customer_upsert")
    .start()
)
```

### Key Point

> Delta's native streaming sink does not directly perform an upsert/`MERGE`; use `foreachBatch` to bridge streaming micro-batches to Delta `MERGE` logic.

---

## 15. `trigger(processingTime=...)`

You can control how frequently streaming data is processed.

Example:

```python
.trigger(processingTime="1 minute")
```

This allows more data to accumulate before a processing cycle.

Instead of:

```text
Event → File
Event → File
Event → File
Event → File
```

you can process a larger micro-batch:

```text
Many events
     ↓
Larger micro-batch
     ↓
Fewer/larger Delta files
```

---

## 16. Why Processing-Time Triggers Can Help Delta

Very frequent streaming writes can create many small Delta files.

```text
Very frequent trigger
        ↓
Many small writes
        ↓
Many small files
```

A larger processing interval can produce:

```text
Longer interval
        ↓
More events accumulated
        ↓
Larger micro-batch
        ↓
Fewer/larger files
```

Potential benefits:

- Fewer files
- Better file sizes
- Better compaction characteristics
- Potentially better query performance

### Trade-Off

The downside is increased latency.

> **Lower latency ↔ more frequent writes**

> **Slightly higher latency ↔ fewer/larger writes**

### Exam Tip

> Use `trigger(processingTime=...)` when a small latency increase is acceptable and you want fewer, larger Delta files.

---

## 17. `availableNow=True`

`availableNow=True` is a batch-like catch-up trigger.

It means:

> Process currently available data and then stop.

Conceptually:

```text
Available data
      ↓
Process everything available
      ↓
STOP
```

### Memory Trick

> **availableNow = Catch up → Stop**

This is useful when you want to process currently available data without keeping the query continuously running.

---

## 18. Continuous Processing

For these notes:

> Continuous processing mode is not the primary Fabric pattern.

The important practical Structured Streaming pattern is the standard micro-batch model.

Do not confuse:

```text
availableNow
```

with:

```text
continuous processing
```

---

## 19. Native Execution Engine

This is an important Fabric-specific exam point.

Do not assume that enabling the Native Execution Engine automatically accelerates every Spark workload.

According to these notes:

> **The Native Execution Engine does not accelerate Structured Streaming.**

Evaluate it for applicable **batch workloads**, not as a Structured Streaming acceleration mechanism.

### Exam Trap

If a question says:

> Enable Native Execution Engine to accelerate Spark Structured Streaming.

Based on these notes:

**Do not choose it.**

### Memory Trick

> **Native Execution Engine ≠ Structured Streaming acceleration**

---

## 20. Complete Streaming Pipeline

A realistic streaming pipeline can look like:

```text
Streaming Source
      ↓
  readStream
      ↓
 Transformations
      ↓
withWatermark()
      ↓
dropDuplicates()
      ↓
 writeStream
      ↓
 foreachBatch
      ↓
    MERGE
      ↓
 Delta Table
```

The streaming query should include:

```text
checkpointLocation
outputMode
```

And optionally:

```text
trigger(processingTime=...)
```

depending on latency and file-size requirements.

---

## 21. The Six Important Questions

### 1. Where did the query stop?

→ `checkpointLocation`

### 2. How long should old streaming state be retained?

→ `withWatermark()`

### 3. What should the sink receive?

→ `outputMode()`

### 4. How do I perform an upsert?

→ `foreachBatch` + `MERGE`

### 5. How frequently should processing happen?

→ `trigger(processingTime=...)`

### 6. Will Native Execution Engine accelerate Structured Streaming?

→ **No, according to these notes.**

---

## 22. DP-700 Exam Decision Table

| Requirement | Answer |
|---|---|
| Track streaming progress/restart | `checkpointLocation` |
| Separate state between queries | Dedicated checkpoint |
| Bound deduplication state | `withWatermark()` |
| Bound windowed aggregation state | `withWatermark()` |
| No revisions / no aggregation | `append` |
| Only changed aggregation rows | `update` |
| Entire aggregation result | `complete` |
| Streaming Delta upsert | `foreachBatch` + `MERGE` |
| Reduce frequency of Delta writes | `trigger(processingTime=...)` |
| Catch up on available data and stop | `availableNow=True` |
| Accelerate Structured Streaming with Native Execution Engine | **No** |

---

## 23. Ultimate Memory Tricks

### Checkpoint

> **Checkpoint = Where did I stop?**

### Watermark

> **Watermark = How long do I remember state?**

### Output Mode

> **Append = New**

> **Update = Changed**

> **Complete = Everything**

### MERGE

> **Streaming MERGE = `foreachBatch`**

### Trigger

> **Trigger = How often do I process/write?**

### availableNow

> **availableNow = Catch up → Stop**

### Native Execution Engine

> **Native Execution Engine ≠ Structured Streaming acceleration**

---

## 24. Most Important Exam Facts

Remember these:

```text
1. Every streaming query → dedicated checkpointLocation

2. dropDuplicates / windowed aggregation
   → withWatermark()

3. Streaming Delta upsert
   → foreachBatch + MERGE

4. append = new/final rows
   update = changed rows
   complete = entire result

5. availableNow = process available data and stop

6. processingTime trigger
   → fewer/larger writes
   → potentially better Delta file sizes
   → slightly more latency

7. Native Execution Engine
   → does NOT accelerate Structured Streaming
```

---

## 25. One-Page Revision

```text
              STRUCTURED STREAMING
                       │
       ┌───────────────┼────────────────┐
       ▼               ▼                ▼
  CHECKPOINT        WATERMARK       OUTPUT MODE
       │               │                │
  Progress/state    Bound state     Append/Update/
                                     Complete
       │
       ▼
   foreachBatch
       │
       ▼
     MERGE
       │
       ▼
   DELTA TABLE
```

### Final Mental Model

```text
Checkpoint → Where did I stop?

Watermark → How long do I remember state?

Output mode → What results should I output?

foreachBatch → How do I run MERGE/upsert?

Trigger → How frequently should I process/write?

availableNow → Catch up and stop.

Native Execution Engine → Not for Structured Streaming acceleration.
```

---

## 26. Real-World Data Engineering Value

These concepts are useful beyond DP-700.

They represent important Data Engineering concepts:

- **Checkpointing** → fault tolerance and recovery
- **Watermarks** → state management and late-arriving data
- **Output modes** → streaming result semantics
- **foreachBatch + MERGE** → streaming upserts
- **Processing-time triggers** → latency vs storage/file-size trade-offs
- **Idempotent processing** → reliable retries and recovery

These ideas transfer to:

- Microsoft Fabric
- Azure Databricks
- Apache Spark
- Structured Streaming
- Delta Lake
- Event-driven data pipelines
