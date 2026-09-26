# DP-700 — Streaming Loading Patterns Notes

## 1. Medallion Architecture for Streaming

The Medallion Architecture applies to streaming data continuously:

| Layer | Purpose | Example |
|---|---|---|
| **Bronze** | Raw, append-only landing layer | Store incoming meter-reading events as received |
| **Silver** | Cleaned, deduplicated, conformed data | Remove duplicate events and standardize records |
| **Gold** | Business-level data and aggregations | Calculate hourly electricity consumption by city |

**Memory trick:** Bronze = Raw → Silver = Clean/Deduplicate → Gold = Business/Aggregate.

Best practice: Keep Bronze append-only and unopinionated. Push transformations, deduplication, and upserts to Silver or Gold.

---

## 2. Streaming Delta Writes and Output Modes

Streaming writes to Delta default to:

```python
.outputMode("append")
```

- **Append:** Write newly produced rows.
- **Update:** Write rows updated since the previous trigger, where supported.
- **Complete:** Write the full result table for each trigger, where supported.

In this pattern, streaming writes land data using append. Upserts and aggregations are handled downstream, for example with `foreachBatch` + `MERGE`.

**Exam tip:** Do not assume a streaming Delta write performs a `MERGE` automatically.

---

## 3. Deduplication: `dropDuplicates()` + `withWatermark()`

### `dropDuplicates()`
Removes repeated records based on selected columns.

### `withWatermark()`
Sets an event-time threshold that helps Spark manage state and handle late-arriving data.

Example:

```python
deduped_df = (
    streaming_df
    .withWatermark("eventTime", "10 minutes")
    .dropDuplicates(["EventID"])
)
```

- `withWatermark("eventTime", "10 minutes")` sets a 10-minute event-time watermark.
- `dropDuplicates(["EventID"])` deduplicates using the event ID while Spark manages state based on the watermark.

**Why pair them?** Streaming deduplication without a watermark can cause state to grow without bound.

**Exam rule:** Pair streaming `dropDuplicates()` with `withWatermark()` to bound state growth.

---

## 4. Fixed `processingTime` Trigger

A processing-time trigger controls how often Spark checks for and processes available data.

Example:

```python
query = (
    streaming_df.writeStream
    .format("delta")
    .outputMode("append")
    .trigger(processingTime="10 seconds")
    .option("checkpointLocation", checkpoint_path)
    .start(output_path)
)
```

A fixed trigger can reduce excessive small-file writes when a small latency tradeoff is acceptable.

**Memory trick:** Fixed trigger → Control processing frequency → Help avoid excessive small files.

---

## 5. Checkpointing, Exactly-Once, and Redelivery

### At-least-once
A source may deliver the same event more than once.

### Exactly-once
In the study notes, exactly-once is scoped to the checkpoint-to-sink relationship, not the entire pipeline.

Checkpointing supports progress tracking and recovery, but it does not automatically prevent an upstream source from redelivering events.

**Best practice:** If the source can redeliver events, add an explicit deduplication step.

**Memory trick:**
- Checkpoint = Processing progress and recovery.
- Deduplication = Remove repeated events.
- Exactly-once = Scoped guarantee, not automatically end-to-end.

---

## 6. Eventhouse Deduplication with `arg_max()`

In Eventhouse, query-time deduplication can use `arg_max()` to select the latest record for each key.

Example:

```kusto
MeterReadings
| summarize arg_max(EventTime, *) by MeterID
```

This returns the record with the maximum `EventTime` for each `MeterID`.

**Important exam point:** Prefer query-time deduplication with `arg_max()` over materialized views when the source is a query-acceleration-enabled OneLake shortcut, because materialized views are not supported there.

---

## 7. Eventstream Destinations

According to the study notes, Eventstream can land streaming data in:
- A Lakehouse
- An Eventhouse

The engine-choice decision matrix is covered separately in the study material.

---

## 8. Best Practices — Quick Checklist

- [ ] Keep Bronze raw and append-only.
- [ ] Perform transformation, deduplication, and upserts in Silver/Gold.
- [ ] Pair streaming `dropDuplicates()` with `withWatermark()`.
- [ ] Use a fixed `processingTime` trigger when a small latency tradeoff is acceptable and excessive small files are a concern.
- [ ] Treat exactly-once as scoped to checkpoint-to-sink behavior.
- [ ] Add explicit deduplication when the source can redeliver events.
- [ ] Use downstream `foreachBatch` + `MERGE` for streaming upserts when appropriate.
- [ ] Prefer query-time `arg_max()` for query-acceleration-enabled OneLake shortcuts rather than unsupported materialized views.

---

## 9. Exam Tips

1. **Streaming Medallion:** Bronze = raw append-only; Silver = deduplicated/conformed; Gold = aggregated/business-level.
2. **Delta streaming default:** `outputMode("append")`.
3. **Deduplication:** `dropDuplicates()` needs a preceding `withWatermark()` to bound state.
4. **Eventhouse:** Materialized views can deduplicate with `arg_max`-style summarization, but are not supported on query-acceleration-enabled OneLake shortcuts.
5. **Exactly-once:** Checkpointing provides a checkpoint-to-sink guarantee, not an end-to-end guarantee. Upstream at-least-once redelivery still needs explicit deduplication.

---

## 10. One-Minute Revision

- **Bronze → Silver → Gold**
- **Bronze → Raw + Append-only**
- **Silver → Clean + Deduplicate**
- **Gold → Aggregate + Business-level**
- **Streaming Delta → Append by default**
- **Deduplication → `withWatermark()` + `dropDuplicates()`**
- **Fixed trigger → Manage micro-batch frequency**
- **Checkpoint → Progress/recovery**
- **Exactly-once → Checkpoint-to-sink scope**
- **Redelivery → Explicit dedup**
- **Accelerated OneLake shortcut → Query-time `arg_max()`**
