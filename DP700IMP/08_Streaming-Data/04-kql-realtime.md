# DP-700 — Eventhouse Ingestion, Shortcuts, Query Acceleration & OneLake Availability

## 1. Streaming Ingestion vs Queued Ingestion

These are two ways of getting data into an Eventhouse/KQL database.

The main trade-off is:

> **Latency vs throughput/reliability**

### Streaming Ingestion

Streaming ingestion is designed for **small, frequent writes where low latency matters**.

Conceptually:

```text
Source
  ↓
Streaming ingestion
  ↓
KQL table
  ↓
Query / Analytics
```

Use streaming ingestion when:

- Data arrives continuously.
- Individual writes are relatively small.
- You need near-real-time availability.
- Many tables may receive frequent small writes.
- Low latency is more important than maximum throughput.

### Example

IoT devices continuously send:

```text
Device 1 → temperature
Device 2 → temperature
Device 3 → temperature
...
```

If the dashboard needs to see those events almost immediately, streaming ingestion makes sense.

---

## 2. Queued Ingestion

Queued ingestion is generally the **default choice for high-volume production workloads**.

Data can be staged/batched and processed efficiently:

```text
Large volume of data
        ↓
Queue / staging
        ↓
Batch processing
        ↓
KQL table
```

Use queued ingestion when:

- Data volume is high.
- Maximum throughput is important.
- Slightly higher latency is acceptable.
- You want reliable, efficient batch-style ingestion.

### Example

A company generates:

```text
500 GB/day
```

of business data and does not require every record to be immediately available.

Queued ingestion is generally more appropriate.

### Streaming vs Queued

| Feature | Streaming | Queued |
|---|---|---|
| Main goal | Low latency | High throughput |
| Data pattern | Small/frequent | Larger batches |
| Availability | Near-real-time | Batch-oriented |
| Production default | Not generally | **Yes** |
| Best for | Latency-sensitive workloads | High-volume workloads |

### Memory Trick

> **Streaming = SPEED**  
> **Queued = SCALE**

Or:

> **Small + frequent + urgent → Streaming**

> **Large + high-volume → Queued**

---

# 3. Query Acceleration

A **shortcut** references data rather than behaving as a native table owned directly by the database.

Conceptually:

```text
Native table
     ↓
Owned directly by database
```

versus:

```text
Shortcut
     ↓
Reference external/existing data
```

A shortcut may have more query overhead than a native table.

**Query acceleration** improves query performance for a shortcut.

```text
Shortcut
   ↓
Query acceleration
   ↓
Performance closer to native table
```

The key idea:

> **Query acceleration closes much of the performance gap between shortcuts and native tables.**

According to the study notes, query acceleration is **generally available (GA)**.

---

# 4. Why Use a Shortcut?

A shortcut lets you reference existing data without unnecessarily duplicating the underlying storage.

Conceptually:

```text
Existing data
     ↓
Shortcut
     ↓
Eventhouse / KQL workload
```

This can be useful when data already exists elsewhere and you want to consume it without creating another physical copy.

---

# 5. Native Table vs Shortcut vs Accelerated Shortcut

Think of three levels:

```text
Native table
     ↓
Owned + directly managed
     ↓
Native/full KQL capabilities
```

```text
Unaccelerated shortcut
     ↓
Referenced data
     ↓
More query overhead
```

```text
Accelerated shortcut
     ↓
Referenced data + acceleration
     ↓
Query performance closer to native
```

| Type | Data relationship | Query performance |
|---|---|---|
| Native table | Owned directly | Best/native |
| Shortcut | Referenced | Lower |
| Accelerated shortcut | Referenced + accelerated | Closer to native |

## Critical Exam Point

> **Query acceleration does NOT turn a shortcut into a native table.**

An accelerated shortcut is still a shortcut.

---

# 6. Materialized Views

A **materialized view** stores a query/transformation result so that it can be queried efficiently without repeatedly calculating the same result.

For example:

```text
Raw Sales
   ↓
GROUP BY Product
   ↓
SUM(Amount)
   ↓
Materialized result
```

Instead of repeatedly calculating the aggregation, the materialized result can be maintained.

## Materialized Views Require Native Tables

This is a major exam restriction:

> **Materialized views require native tables.**

Even if a shortcut has query acceleration enabled, it is still not a native table.

Therefore:

```text
Accelerated shortcut
        ↓
Materialized view
        ❌
```

---

# 7. Update Policies

An **update policy** is a KQL transformation mechanism.

Conceptually:

```text
Source table
     ↓
Update Policy
     ↓
Transformation
     ↓
Target table
```

For example:

```text
RawEvents
    ↓
Update Policy
    ↓
CleanEvents
```

When data arrives in the source table, the update policy can automatically transform it into another table.

## Update Policies Require Native Tables

Another important exam restriction:

> **Update policies require native tables.**

Therefore:

```text
Native table
   ↓
Update policy
   ↓
Target
```

is supported.

But:

```text
Accelerated shortcut
   ↓
Update policy
```

is not equivalent to using a native table.

---

# 8. Materialized View vs Update Policy

Both have the same important exam constraint:

| Mechanism | Native table required? |
|---|---:|
| Materialized view | **Yes** |
| Update policy | **Yes** |
| Accelerated shortcut | **No — it remains a shortcut** |

### Memory Trick

> **Materialized View → Native**

> **Update Policy → Native**

> **Acceleration → Faster Shortcut, not Native**

---

# 9. OneLake Availability

**OneLake availability** makes Eventhouse/KQL database data available through OneLake.

Conceptually:

```text
Eventhouse / KQL database
          ↓
   OneLake availability
          ↓
      Delta files
          ↓
 ┌────────┼─────────┐
 ↓        ↓         ↓
Spark   Warehouse  Power BI
```

The key idea:

> **Eventhouse data → OneLake → broader Fabric consumption**

This allows other Fabric workloads to consume the data.

---

# 10. Why OneLake Availability?

Imagine event data is stored in Eventhouse.

Different teams may want to consume it using:

- Spark
- Warehouse
- Power BI

OneLake availability exposes the data through OneLake as **Delta Lake files**, allowing broader Fabric consumption.

Conceptually:

```text
Eventhouse
     ↓
OneLake availability
     ↓
Delta files
     ↓
Fabric analytics ecosystem
```

---

# 11. Database-Level OneLake Availability

OneLake availability can be enabled at the **database level**.

Choose this when:

> **Most or all tables in the database should be broadly available through OneLake.**

Example:

```text
KQL Database
├── Table A
├── Table B
├── Table C
├── Table D
└── Table E

        ↓

Database-level OneLake availability

        ↓

Most/all tables exposed
```

### Memory

> **Database level = Broad exposure**

---

# 12. Table-Level OneLake Availability

You can also enable OneLake availability for an **individual table**.

Choose this when only selected tables should be exposed.

Example:

```text
KQL Database

Customers       ❌
Transactions    ✅
Telemetry       ❌
Orders          ✅
Logs            ❌
```

### Memory

> **Table level = Selective exposure**

---

# 13. Database-Level vs Table-Level

| Requirement | Choose |
|---|---|
| Most/all tables should be available | **Database level** |
| Only selected tables should be available | **Table level** |

### Simple Memory Trick

> **Broad → Database**

> **Selective → Table**

---

# 14. Adaptive Batching

OneLake availability does **not necessarily mean immediate visibility** of data in OneLake.

It uses **adaptive batching**.

Conceptually:

```text
Eventhouse receives data
        ↓
Data accumulates
        ↓
Adaptive batching
        ↓
Delta files in OneLake
```

Therefore:

> **A delay does not automatically mean OneLake availability has failed.**

The system can batch data before making it available in OneLake.

---

# 15. Why Is There a Delay?

The system balances:

```text
Low latency
     ↕
Efficient batching
```

If every tiny record immediately created file operations, processing could become inefficient.

Batching allows more efficient writes.

Therefore, data may take time to appear in OneLake by design.

---

# 16. Target Latency

According to the study notes:

- OneLake availability's adaptive batching can delay data appearing in OneLake by **up to 3 hours by default**.
- `TargetLatencyInMinutes` can be used to tune the desired latency.
- The notes specify a **5-minute floor**.

Conceptually:

```text
Default target
     ↓
Up to 3 hours

Can tune lower
     ↓
TargetLatencyInMinutes
     ↓
5-minute floor
```

### Exam Memory

> **Don't expect instant OneLake visibility.**

---

# 17. Troubleshooting OneLake Availability

If data is not immediately visible in OneLake, don't immediately assume that the feature has failed.

Check:

```kusto
.show table mirroring operations
```

This helps inspect mirroring/availability operations.

Conceptually:

```text
Data not visible
      ↓
Don't immediately assume failure
      ↓
Check mirroring operations
      ↓
Understand batching/latency state
```

### Important Exam/Operational Point

> **Check `.show table mirroring operations` before assuming OneLake availability has failed.**

---

# 18. What Is Exposed to OneLake?

The Eventhouse/KQL data becomes available as **Delta Lake files**.

Conceptually:

```text
KQL / Eventhouse
      ↓
OneLake availability
      ↓
Delta format
      ↓
Fabric workloads
```

The study notes identify these consumers:

- Spark
- Warehouse
- Power BI

---

# 19. OneLake Availability and Retention

OneLake availability is governed by the **KQL database's own retention**.

Think:

```text
KQL Database retention
          ↓
Data lifecycle
          ↓
OneLake availability
```

So OneLake availability should not be thought of as an independent unlimited-history store.

---

# 20. Restrictions When OneLake Availability Is Enabled

This is **very important for DP-700**.

According to the notes, the following operations are blocked while OneLake availability is enabled:

### 1. Table Rename

```text
OldTable → NewTable
```

❌ Blocked.

### 2. Column-Type Changes

Example:

```text
CustomerID INT
```

to:

```text
CustomerID STRING
```

❌ Blocked.

### 3. Row-Level Security

Row-level security is:

❌ Blocked.

### 4. Delete

Deleting data is:

❌ Blocked.

### 5. Truncate

Truncating the table is:

❌ Blocked.

### 6. Purge

Purging data is:

❌ Blocked.

---

# 21. OneLake Restrictions — Memory Table

| Operation | OneLake availability enabled |
|---|---:|
| Rename table | ❌ Blocked |
| Change column type | ❌ Blocked |
| Row-level security | ❌ Blocked |
| Delete | ❌ Blocked |
| Truncate | ❌ Blocked |
| Purge | ❌ Blocked |

### Memory Trick

> **OneLake availability restricts structure + security + destructive operations.**

---

# 22. Complete Architecture

Putting the concepts together:

```text
                    IoT / Data Sources
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
        Streaming                   Queued
              │                         │
        Low latency              High volume
              │                         │
              └────────────┬────────────┘
                           ▼
                      KQL Tables
                           │
             ┌─────────────┼──────────────┐
             │             │              │
             ▼             ▼              ▼
          Native       Shortcut       Shortcut
           Table                       + Query
                                      Acceleration
             │
       ┌─────┴─────┐
       ▼           ▼
 Update Policy   Materialized View
       │           │
       └─────┬─────┘
             ▼
          Analytics

KQL Database
      │
      ▼
OneLake Availability
      │
      ▼
Adaptive Batching
      │
      ▼
Delta Files in OneLake
      │
      ├── Spark
      ├── Warehouse
      └── Power BI
```

---

# 23. Most Important Relationships

## Ingestion

```text
Streaming
↓
Small + frequent + latency-sensitive
```

```text
Queued
↓
High volume + throughput + production default
```

---

## Shortcuts

```text
Shortcut
↓
Reference existing data
```

```text
Query acceleration
↓
Improve shortcut query performance
```

```text
Accelerated shortcut
≠
Native table
```

---

## Native-Table-Only Mechanisms

```text
Materialized View
↓
Native table required
```

```text
Update Policy
↓
Native table required
```

---

## OneLake

```text
Eventhouse
    ↓
OneLake availability
    ↓
Delta files
    ↓
Spark / Warehouse / Power BI
```

---

# 24. Exam Scenarios

## Scenario 1

A company receives millions of records and prioritizes throughput over immediate availability.

**Answer:**

✅ **Queued ingestion**

---

## Scenario 2

Small events arrive frequently from many sources and need near-real-time availability.

**Answer:**

✅ **Streaming ingestion**

---

## Scenario 3

A shortcut has poor query performance. You want performance closer to a native table without duplicating storage.

**Answer:**

✅ **Query acceleration**

---

## Scenario 4

You enabled query acceleration on a shortcut and want to create a materialized view.

**Answer:**

❌ **Not supported**

Reason:

> Query acceleration does not turn a shortcut into a native table.

---

## Scenario 5

You want to apply an update policy to an accelerated shortcut.

**Answer:**

❌ **Not supported**

Reason:

> Update policies require native tables.

---

## Scenario 6

Almost every table in a KQL database should be consumable through OneLake.

**Answer:**

✅ **Database-level OneLake availability**

---

## Scenario 7

Only two tables should be exposed through OneLake.

**Answer:**

✅ **Table-level OneLake availability**

---

## Scenario 8

OneLake availability is enabled, but the data has not appeared immediately.

Should you assume failure?

**Answer:**

❌ **No**

Adaptive batching can introduce delay.

Check:

```kusto
.show table mirroring operations
```

---

## Scenario 9

You want to tune OneLake availability latency.

**Answer:**

Use:

```text
TargetLatencyInMinutes
```

The study notes specify a **5-minute floor**.

---

## Scenario 10

OneLake availability is enabled and you want to rename a table.

**Answer:**

❌ **Blocked**

---

# 25. Final DP-700 Memory Sheet

## Ingestion

```text
STREAMING
↓
Small + frequent + latency-sensitive
```

```text
QUEUED
↓
High volume + throughput + production default
```

---

## Shortcuts

```text
SHORTCUT
↓
Reference existing data
```

```text
QUERY ACCELERATION
↓
Faster shortcut queries
↓
Generally available (GA)
↓
Still NOT a native table
```

---

## Native-Only

```text
MATERIALIZED VIEW
↓
Native table only
```

```text
UPDATE POLICY
↓
Native table only
```

---

## OneLake Availability

```text
EVENTHOUSE
↓
OneLake availability
↓
Delta files
↓
Spark / Warehouse / Power BI
```

---

## Scope

```text
DATABASE LEVEL
↓
Broad exposure
```

```text
TABLE LEVEL
↓
Selective exposure
```

---

## Latency

```text
ADAPTIVE BATCHING
↓
OneLake visibility may be delayed
↓
Up to 3 hours default target
↓
TargetLatencyInMinutes
↓
5-minute floor
```

---

## Troubleshooting

```kusto
.show table mirroring operations
```

Use this before assuming OneLake availability has failed.

---

## Restrictions

```text
OneLake availability enabled
        ↓
❌ Rename table
❌ Change column types
❌ Row-level security
❌ Delete
❌ Truncate
❌ Purge
```

---

# 26. Ultimate DP-700 Memory Trick

> **Streaming = FAST**

> **Queued = BIG**

> **Shortcut = REFERENCE**

> **Acceleration = FASTER REFERENCE**

> **Native = FULL KQL CAPABILITIES**

> **Materialized View = NATIVE**

> **Update Policy = NATIVE**

> **OneLake = SHARE EVENTHOUSE DATA**

> **Adaptive batching = NOT INSTANT**

> **Mirroring operations = CHECK STATUS**

> **Database level = BROAD**

> **Table level = SELECTIVE**
