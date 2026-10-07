# DP-700 Microsoft Fabric — Spark, Warehouse & COPY INTO Troubleshooting Notes

## Overview

This note covers the supplied DP-700 study points around:

- Spark troubleshooting and exit codes
- Spark configuration with `%%configure` and `spark.conf.set()`
- Fabric Spark capacity and queueing
- Fabric Warehouse snapshot isolation
- Warehouse write-write conflicts
- Retry with backoff
- `COPY INTO`, `ERRORFILE`, and `MAXERRORS`
- Datetime `CORRECTED` rebase mode
- Power Query error families
- Capacity/throttling errors
- Query folding
- `AnalysisException`
- Unsupported T-SQL features

---

# 1. Spark Troubleshooting — Check Exit Codes First

When a Spark job fails, first check:

```text
Spark UI
   ↓
Executors tab
   ↓
Exit codes
```

Do not immediately start changing every `spark.*` configuration.

The exit code gives an important clue about the type of failure.

### Important exit codes

| Exit code | Meaning | Exam signal |
|---:|---|---|
| **137** | OOM / SIGKILL | Out of memory |
| **143** | SIGTERM | Often benign scale-down |
| **134** | SIGABRT | Process aborted |
| **1** | User code error | Application/code problem |
| **-100** | Preempted | Compute was preempted |

### Memory trick

> **137 = Memory**  
> **143 = Terminated**  
> **134 = Abort**  
> **1 = User code**  
> **-100 = Preempted**

---

# 2. Exit Code 137 — OOM

Exit code:

```text
137
```

means:

> **Out of Memory (OOM) / SIGKILL**

The next question is:

> Where did the memory problem occur?

It can be a **driver-side** or **executor-side** problem.

These require different investigation paths.

---

# 3. Driver-Side OOM

The Spark driver coordinates the Spark application.

A common driver-memory problem occurs when too much data is brought back to the driver.

Examples:

```python
df.collect()
```

or:

```python
df.toPandas()
```

Conceptually:

```text
Large distributed DataFrame
          ↓
      collect()
          ↓
Data comes to driver
          ↓
Driver memory exhausted
          ↓
OOM
```

### Exam thinking

> **Driver OOM → investigate operations that bring/hold too much data at the driver.**

Examples from the study material:

- `collect()`
- `toPandas()`

---

# 4. Executor-Side OOM

Executors perform distributed Spark work.

An executor can run out of memory when it receives too much work/data.

Important investigation areas:

- Data skew
- Partitioning
- Caching

### Example: Data skew

```text
Partition 1 → 10 MB
Partition 2 → 12 MB
Partition 3 → 11 MB
Partition 4 → 900 GB
```

One executor processing the huge partition can run out of memory.

```text
Data skew
   ↓
Huge partition
   ↓
Executor receives excessive data
   ↓
Executor OOM
```

### Memory

> **Driver OOM → collect/toPandas-type investigation**  
> **Executor OOM → skew/partitioning/caching investigation**

---

# 5. Why Check Exit Code Before Changing Spark Configuration?

Suppose you see:

```text
Exit code = 143
```

and immediately increase:

```text
spark.executor.memory
```

You may be solving the wrong problem.

Exit code `143` means:

```text
SIGTERM
```

and can often be associated with benign scale-down behavior according to the supplied study material.

### Better approach

```text
Job fails
   ↓
Check exit code
   ↓
Understand failure
   ↓
Choose appropriate fix
   ↓
Change configuration only if justified
```

### Exam takeaway

> **Diagnose first. Tune Spark configuration second.**

---

# 6. `%%configure` vs `spark.conf.set()`

This is an important DP-700 distinction.

## `%%configure`

Use a `%%configure` first cell for settings such as:

```text
spark.executor.*
spark.driver.*
network settings
YARN settings
```

Conceptually:

```text
%%configure
     ↓
Spark/application configuration
     ↓
Session starts with those settings
```

---

# 7. `spark.conf.set()`

Use:

```python
spark.conf.set(...)
```

for runtime-tunable settings, especially:

```text
spark.sql.*
```

Example:

```python
spark.conf.set("spark.sql.some.setting", "value")
```

### Memory

> **`%%configure` → executor/driver/network/YARN/application-level setup**

> **`spark.conf.set()` → runtime-tunable `spark.sql.*` settings**

---

# 8. Spark Configuration Exam Pattern

If the question asks where to configure:

```text
spark.executor.*
spark.driver.*
network
YARN
```

Think:

> **`%%configure` in the first cell**

If it asks about runtime-tunable:

```text
spark.sql.*
```

Think:

> **`spark.conf.set()`**

---

# 9. Fabric Spark Capacity

Spark workloads require Fabric capacity.

When capacity is exhausted, behavior depends on the type of workload.

There are two important cases:

```text
Scheduled / Pipeline jobs
```

and:

```text
Interactive notebook runs
```

---

# 10. `TooManyRequestsForCapacity`

Important error:

```text
TooManyRequestsForCapacity
```

This corresponds to:

```text
HTTP 430
```

Think:

> **Capacity pressure / throttling**

It does not automatically mean that your Spark code is wrong.

### Memory

> **TooManyRequestsForCapacity = HTTP 430 = capacity problem**

---

# 11. Scheduled/Pipeline Spark Jobs

When capacity is unavailable, scheduled/pipeline jobs can be queued.

Conceptually:

```text
Pipeline/Scheduled Spark job
          ↓
Capacity unavailable
          ↓
Queue
          ↓
FIFO
          ↓
Run when capacity becomes available
```

### FIFO

FIFO means:

> **First In, First Out**

Earlier queued jobs are processed before later ones according to the queueing model.

---

# 12. 24-Hour Queue Expiration

Queued jobs do not wait forever.

The supplied study material states:

> Queued jobs expire after **24 hours**.

Conceptually:

```text
Job submitted
     ↓
Capacity unavailable
     ↓
Queued
     ↓
Wait
     ↓
24 hours
     ↓
Expires
```

### Exam memory

> **Queued Spark job → 24-hour expiry**

---

# 13. Interactive Notebook Runs

Interactive notebook runs behave differently.

According to the supplied material:

> Interactive notebook runs **do not queue**.

If capacity is unavailable:

```text
Interactive notebook
       ↓
Capacity unavailable
       ↓
Immediate failure
```

### Comparison

| Workload | Capacity unavailable |
|---|---|
| Scheduled/pipeline Spark job | Queues |
| Interactive notebook run | Fails immediately |
| Queue expiry | 24 hours |

### Exam signal

> **Interactive = no queue**

---

# 14. Fabric Warehouse — Snapshot Isolation

Fabric Warehouse uses:

> **Snapshot isolation only**

This is a major exam fact.

Conceptually, snapshot isolation gives a transaction a consistent view/snapshot of the data.

For the exam, the key point is:

```text
Fabric Warehouse
       ↓
Snapshot isolation only
```

---

# 15. `SET TRANSACTION ISOLATION LEVEL`

Do not assume you can change Fabric Warehouse's isolation level using:

```sql
SET TRANSACTION ISOLATION LEVEL ...
```

The supplied study material states that this is:

> **Silently ignored**

### Exam memory

> **Fabric Warehouse = snapshot isolation only**

> **`SET TRANSACTION ISOLATION LEVEL` = ignored**

---

# 16. Write-Write Conflicts

Suppose two operations try to write to the same Warehouse table at the same time.

```text
Transaction A ──┐
                ├──> Same table
Transaction B ──┘
```

A write-write conflict can occur.

The supplied material identifies:

```text
24556
24706
```

as write-write conflict errors.

---

# 17. Handling Write-Write Conflicts

The recommended solution is:

> **Retry with backoff**

Conceptually:

```text
Write
  ↓
Conflict
  ↓
24556 / 24706
  ↓
Wait
  ↓
Retry
  ↓
Success
```

The important recommendation is to build this retry behavior into:

- Applications
- Pipeline stored procedures

when they write to Fabric Warehouse tables under concurrent load.

---

# 18. What Is Retry With Backoff?

Do not continuously retry without waiting.

Bad pattern:

```text
Fail
 ↓
Retry immediately
 ↓
Fail
 ↓
Retry immediately
 ↓
Fail
```

Better:

```text
Fail
 ↓
Wait
 ↓
Retry
 ↓
If needed, wait longer
 ↓
Retry
```

Example:

```text
Attempt 1 → Fail
Wait 1 second

Attempt 2 → Fail
Wait 2 seconds

Attempt 3 → Fail
Wait 4 seconds

Attempt 4 → Success
```

### Memory

> **Conflict → wait → retry**

---

# 19. Table-Level Evaluation

The supplied material emphasizes that write-write conflicts are evaluated at the:

> **Table level**

So when concurrent operations target the same Warehouse table, retry logic is important.

```text
Concurrent writes
       ↓
Same table
       ↓
Write-write conflict
       ↓
Retry with backoff
```

---

# 20. `COPY INTO`

`COPY INTO` is used to load external file data into a Warehouse table.

The supplied study material focuses on:

- CSV
- JSONL

especially when the source is external and less trusted.

---

# 21. Problem: One Bad Row

Imagine:

```text
1,000,000 rows
```

and only one row is malformed:

```text
999,999 → valid
1        → bad
```

Without error-tolerance handling, one bad row can cause the load to fail.

```text
COPY INTO
   ↓
Bad row
   ↓
Load failure
```

For external/less-trusted sources, this may not be desirable.

---

# 22. `MAXERRORS`

`MAXERRORS` controls how many errors can be tolerated during the load.

The supplied material states:

> Default `MAXERRORS` = **0**

So:

```text
MAXERRORS = 0
```

means no load errors are tolerated under the described behavior.

### Memory

> **Default MAXERRORS = 0**

---

# 23. Why Use `MAXERRORS`?

If you know an external source can occasionally contain malformed rows, you can configure an appropriate error tolerance.

Conceptually:

```text
External file
      ↓
COPY INTO
      ↓
Bad rows
      ↓
MAXERRORS
      ↓
Allow configured number of errors
```

The goal is not to blindly ignore bad data.

The goal is to:

1. Prevent one bad row from unnecessarily failing the entire batch.
2. Capture diagnostics about rejected rows.
3. Investigate the bad records separately.

---

# 24. `ERRORFILE`

`ERRORFILE` provides a location for error/rejected-row information.

The supplied material says the output includes:

```text
error.Json
row.csv
```

under a:

```text
statement-ID folder
```

Conceptually:

```text
COPY INTO
    ↓
Bad rows
    ↓
ERRORFILE
    ↓
Statement-ID folder
       ├── error.Json
       └── row.csv
```

This makes the failed records diagnosable.

---

# 25. `ERRORFILE` Restriction

This is an important exam trap.

The supplied material states:

> `ERRORFILE` applies to **CSV and JSONL only**.

Therefore:

| File format | `ERRORFILE` |
|---|---|
| CSV | ✅ |
| JSONL | ✅ |
| Parquet | ❌ |

### Memory

> **ERRORFILE → CSV + JSONL**

---

# 26. `ERRORFILE` + `MAXERRORS`

These two settings work together.

```text
External CSV/JSONL
        ↓
     COPY INTO
        ↓
    MAXERRORS
        ↓
Allow configured bad rows
        ↓
    ERRORFILE
        ↓
Record diagnostic information
```

This can turn:

```text
Hard batch failure
```

into a:

```text
Partial + diagnosable load
```

when the configured error tolerance allows it.

---

# 27. Example: `COPY INTO`

Suppose:

```text
1000 rows
5 bad rows
```

If:

```text
MAXERRORS = 0
```

the bad rows can cause the load to fail.

If you intentionally configure a suitable tolerance:

```text
MAXERRORS = 5
```

then those errors can be tolerated while error details are written to the error output.

### Important

`MAXERRORS` does **not** mean:

> "Bad data doesn't matter."

It means:

> **Allow a configured number of errors and capture their details.**

---

# 28. Datetime Rebase Mode — `CORRECTED`

The supplied study point recommends validating:

```text
CORRECTED
```

datetime rebase mode against historical data before applying it broadly after a runtime upgrade.

Why?

Because historical datetime interpretation can matter when runtime behavior changes.

Conceptually:

```text
Runtime upgrade
      ↓
Historical datetime values
      ↓
Rebase behavior
      ↓
Potential interpretation differences
```

---

# 29. How to Validate `CORRECTED`

Don't immediately apply it to every historical record.

Instead:

```text
Historical sample
       ↓
Apply CORRECTED
       ↓
Compare results
       ↓
Validate dates
       ↓
Apply broadly if correct
```

### Exam memory

> **Runtime upgrade + historical dates → test `CORRECTED` on a sample first.**

---

# 30. Power Query Error Families

Three important Power Query errors:

```text
DataFormat.Error
DataSource.Error
Expression.Error
```

The easiest memory trick:

> **DATA → CONNECTION → LOGIC**

---

# 31. `DataFormat.Error`

Think:

> **Problem with the data's format/shape.**

Conceptually:

```text
Source data
     ↓
Unexpected format/shape
     ↓
DataFormat.Error
```

Examples could involve malformed values or unexpected data types/formats.

### Memory

> **DataFormat = DATA problem**

---

# 32. `DataSource.Error`

Think:

> **Problem connecting to/accessing the source.**

Conceptually:

```text
Power Query
    ↓
Data source
    ↓
Connectivity/access problem
    ↓
DataSource.Error
```

### Memory

> **DataSource = CONNECTION problem**

---

# 33. `Expression.Error`

Think:

> **Problem with Power Query/M logic or expression.**

Conceptually:

```text
M expression
     ↓
Invalid logic/reference/operation
     ↓
Expression.Error
```

### Memory

> **Expression = LOGIC problem**

---

# 34. Error Family Comparison

| Error | Think | Layer |
|---|---|---|
| `DataFormat.Error` | Bad data format/shape | Data |
| `DataSource.Error` | Source/connectivity problem | Connection |
| `Expression.Error` | M/transformation logic problem | Logic |

### Ultimate memory

> **Format = Data**  
> **Source = Connection**  
> **Expression = Logic**

---

# 35. Capacity and Throttling Errors

Not every pipeline/Spark failure is caused by application code.

Some failures are capacity-related.

Important examples:

```text
2003
CapacityLimitExceeded
HTTP 430
```

These point toward capacity pressure/throttling.

Conceptually:

```text
High workload
     ↓
Capacity pressure
     ↓
Throttling / capacity limit
     ↓
Workload failure
```

---

# 36. How to Handle Capacity Errors

When the error indicates capacity pressure, investigate capacity rather than immediately modifying application logic.

The supplied material points to:

> **Capacity Metrics app**

Conceptually:

```text
Capacity-related error
        ↓
Capacity Metrics app
        ↓
Check utilization / throttling
```

### Exam signal

If you see:

```text
2003
CapacityLimitExceeded
HTTP 430
```

think:

> **Capacity problem**

---

# 37. Query Folding

Query folding is an important performance concept.

The basic idea:

> Power Query tries to push transformations back to the data source so that the source performs the work.

Conceptually:

```text
Power Query transformation
          ↓
Can source perform it?
          ↓
Push transformation to source
          ↓
Less data movement
          ↓
Better performance
```

---

# 38. Query Folding Failure Can Be Silent

This is a very important exam concept.

Suppose:

```text
Old refresh = 20 minutes
New refresh = 2 hours
```

but:

```text
No error
```

Do not automatically assume that the system is temporarily slow.

The supplied study point says:

> Treat a refresh-duration regression with no error as a **query-folding investigation**.

Conceptually:

```text
Refresh becomes much slower
        +
No error
        ↓
Investigate query folding
```

---

# 39. Folding vs No Folding

### Folding works

```text
Source
  ↓
Filter/transformation pushed to source
  ↓
Source processes it
  ↓
Smaller result returned
  ↓
Faster
```

### Folding is lost

```text
Source
  ↓
Large dataset returned
  ↓
Power Query processes transformation
  ↓
More data movement/computation
  ↓
Slower
```

### Exam memory

> **Slow but successful → investigate query folding**

---

# 40. `AnalysisException`

`AnalysisException` is a Spark analysis/planning error.

It generally occurs before expensive distributed execution begins.

Conceptually:

```text
Spark code
   ↓
Analyze query/schema
   ↓
Problem detected
   ↓
AnalysisException
   ↓
Execution does not proceed normally
```

The supplied study material emphasizes:

> **AnalysisException fails fast, before compute is spent.**

---

# 41. What Does `AnalysisException` Usually Point Toward?

Its message often identifies the relevant:

- Table
- Column
- Data type
- Query/schema problem

Therefore:

```text
AnalysisException
       ↓
Read the message
       ↓
Check table/column/type
       ↓
Fix query/schema
```

Don't immediately start increasing executor memory.

### Memory

> **AnalysisException = analysis/query/schema problem**

---

# 42. Unsupported T-SQL in Fabric Warehouse

The supplied material identifies these as unsupported:

```text
Triggers
Synonyms
Materialized views
SET ROWCOUNT
SET TRANSACTION ISOLATION LEVEL
Recursive queries
```

---

# 43. Triggers ❌

Don't assume Fabric Warehouse supports traditional SQL triggers such as:

```sql
CREATE TRIGGER ...
```

### Exam signal

If an answer proposes a trigger for automatic Warehouse behavior:

> Treat it as unsupported according to these notes.

---

# 44. Synonyms ❌

Synonyms are also listed as unsupported.

```sql
CREATE SYNONYM ...
```

→ ❌

---

# 45. Materialized Views ❌

Materialized views are listed as unsupported in the supplied Fabric Warehouse notes.

→ ❌

---

# 46. `SET ROWCOUNT` ❌

The notes identify:

```sql
SET ROWCOUNT ...
```

as unsupported.

→ ❌

---

# 47. `SET TRANSACTION ISOLATION LEVEL` ❌

Remember:

```sql
SET TRANSACTION ISOLATION LEVEL ...
```

is listed as unsupported/ignored.

Fabric Warehouse uses:

> **Snapshot isolation only**

---

# 48. Recursive Queries ❌

Recursive query patterns are also listed as unsupported.

### Exam memory list

```text
Triggers
Synonyms
Materialized views
SET ROWCOUNT
SET TRANSACTION ISOLATION LEVEL
Recursive queries
```

---

# 49. Complete Spark Troubleshooting Decision Tree

```text
Spark job fails
      ↓
Spark UI → Executors
      ↓
Check EXIT CODE
      │
      ├── 137
      │    ↓
      │   OOM
      │    ├── Driver → collect/toPandas investigation
      │    └── Executor → skew/partitioning/caching
      │
      ├── 143
      │    ↓
      │   SIGTERM
      │    ↓
      │   Often benign scale-down
      │
      ├── 134
      │    ↓
      │   SIGABRT
      │
      ├── 1
      │    ↓
      │   User code error
      │
      └── -100
           ↓
         Preempted
```

---

# 50. Fabric Spark Capacity Decision Tree

```text
Spark job submitted
       ↓
Capacity unavailable?
       │
       ├── Scheduled/Pipeline job
       │       ↓
       │     Queue
       │       ↓
       │      FIFO
       │       ↓
       │   24h expiry
       │
       └── Interactive notebook
               ↓
         No queue
               ↓
       Immediate failure
```

If you see:

```text
TooManyRequestsForCapacity
HTTP 430
```

think:

> **Capacity pressure / throttling**

---

# 51. Warehouse Concurrency Decision Tree

```text
Concurrent writes
       ↓
Same Warehouse table
       ↓
Write-write conflict
       │
       ├── 24556
       │
       └── 24706
              ↓
       Retry with backoff
```

Remember:

> **Conflict is evaluated at table level.**

---

# 52. `COPY INTO` Decision Tree

```text
External file
     ↓
CSV / JSONL?
     │
     ├── YES
     │    ↓
     │ ERRORFILE
     │    +
     │ MAXERRORS
     │
     └── PARQUET
          ↓
       ERRORFILE
          ❌
```

Default:

```text
MAXERRORS = 0
```

Error output:

```text
statement-ID/
    error.Json
    row.csv
```

---

# 53. Query Folding Decision Tree

```text
Dataflow refresh
       ↓
Refresh much slower?
       ↓
No error?
       ↓
Investigate query folding
       ↓
Check whether transformations
are still pushed to source
```

---

# 54. DP-700 Exam Cheat Sheet

## Spark Exit Codes

| Code | Meaning |
|---:|---|
| **137** | OOM / SIGKILL |
| **143** | SIGTERM; often benign scale-down |
| **134** | SIGABRT |
| **1** | User code error |
| **-100** | Preempted |

### Memory

> **137 = Memory**  
> **143 = Terminated**  
> **134 = Abort**  
> **1 = Code**  
> **-100 = Preempted**

---

## Spark Configuration

```text
spark.executor.*
spark.driver.*
network/YARN
        ↓
%%configure
```

```text
spark.sql.*
        ↓
spark.conf.set()
```

---

## Capacity

```text
TooManyRequestsForCapacity
        ↓
HTTP 430
        ↓
Capacity pressure
```

```text
Scheduled/Pipeline
        ↓
Queue → FIFO → 24h expiry
```

```text
Interactive Notebook
        ↓
No queue → immediate failure
```

---

## Fabric Warehouse

```text
Isolation
    ↓
Snapshot only
```

```text
24556 / 24706
       ↓
Write-write conflict
       ↓
Retry with backoff
```

---

## COPY INTO

```text
ERRORFILE
   ↓
CSV + JSONL only
```

```text
MAXERRORS
   ↓
Controls tolerated errors
```

Default:

```text
MAXERRORS = 0
```

Output:

```text
statement-ID/
    error.Json
    row.csv
```

---

## Power Query Errors

```text
DataFormat.Error
        ↓
DATA

DataSource.Error
        ↓
CONNECTION

Expression.Error
        ↓
LOGIC
```

### Memory

> **DATA → CONNECTION → LOGIC**

---

## Query Folding

```text
Refresh slower
+
No error
        ↓
Query folding investigation
```

---

## Datetime Rebase

```text
Runtime upgrade
+
Historical dates
        ↓
Test CORRECTED
against a sample
before applying broadly
```

---

## Unsupported T-SQL

```text
Triggers                       ❌
Synonyms                       ❌
Materialized views             ❌
SET ROWCOUNT                   ❌
SET TRANSACTION ISOLATION LEVEL ❌
Recursive queries              ❌
```

---

# 55. Key Takeaways

- **Check Spark exit codes before changing `spark.*` configuration.**
- **137 = OOM/SIGKILL.**
- Driver-side and executor-side OOM require different investigations.
- `%%configure` is for `spark.executor.*`, `spark.driver.*`, network, and YARN-style settings in the supplied guidance.
- `spark.conf.set()` is for runtime-tunable `spark.sql.*` settings.
- `TooManyRequestsForCapacity` = **HTTP 430** and indicates capacity pressure.
- Scheduled/pipeline Spark jobs can queue; queued jobs expire after **24 hours**.
- Interactive notebook runs **do not queue**.
- Fabric Warehouse uses **snapshot isolation only**.
- Write-write conflicts **24556** and **24706** should be handled with **retry + backoff**.
- These conflicts are evaluated at the **table level**.
- `COPY INTO` with `ERRORFILE`/`MAXERRORS` is useful for external, less-trusted CSV/JSONL sources.
- `ERRORFILE` applies to **CSV and JSONL only**, not Parquet.
- Default `MAXERRORS` is **0**.
- `CORRECTED` datetime rebase mode should be validated against historical samples before broad application after a runtime upgrade.
- `DataFormat.Error` = data/format.
- `DataSource.Error` = connectivity/source.
- `Expression.Error` = M/logic.
- `2003`, `CapacityLimitExceeded`, and HTTP `430` point toward capacity/throttling.
- A refresh that becomes much slower without an error should trigger a **query-folding investigation**.
- `AnalysisException` fails fast during analysis rather than after expensive compute.
- Memorize the unsupported T-SQL list supplied above.

---

# 56. Ultimate Memory Sheet

```text
SPARK
 ↓
Check exit code FIRST

137  → OOM
143  → SIGTERM
134  → SIGABRT
1    → User code
-100 → Preempted
```

```text
SPARK CONFIG
 ↓
%%configure → executor/driver/network/YARN
spark.conf.set() → runtime spark.sql.*
```

```text
CAPACITY
 ↓
TooManyRequestsForCapacity
 ↓
HTTP 430
 ↓
Capacity pressure
```

```text
SPARK QUEUE
 ↓
Scheduled/Pipeline → Queue → FIFO → 24h expiry
Interactive        → No queue → Immediate failure
```

```text
WAREHOUSE
 ↓
Snapshot isolation only
 ↓
24556 / 24706
 ↓
Write-write conflict
 ↓
Retry + backoff
```

```text
COPY INTO
 ↓
CSV / JSONL
 ↓
ERRORFILE + MAXERRORS
 ↓
Diagnosable partial load
```

```text
POWER QUERY
 ↓
DataFormat  → DATA
DataSource  → CONNECTION
Expression  → LOGIC
```

```text
SLOW REFRESH + NO ERROR
 ↓
QUERY FOLDING
```

```text
ANALYSISEXCEPTION
 ↓
FAILS FAST
 ↓
Check table / column / type / query
```

> **Final exam mindset: Diagnose the layer first, then choose the fix.**
