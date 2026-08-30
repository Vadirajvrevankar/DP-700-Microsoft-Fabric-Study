# DP-700 – Missing Concepts / Final Theory Completion

> **Purpose:** This document contains the important DP-700 theory topics that were missing or not covered deeply enough in Lessons 1–5.
>
> It is designed to complete the **theoretical coverage** of the current DP-700 skills measured as of **July 21, 2026**.
>
> It does not replace hands-on practice, but it closes the major theory gaps identified against the official Microsoft study guide.

---

# 1. Configure Fabric Workspace Settings

## 1.1 Spark Workspace Settings

Fabric workspaces can have Spark-related settings that control how Spark workloads behave in that workspace.

Important ideas:

- Spark compute configuration
- Spark pool/settings
- Default Spark environment/settings
- Resource configuration
- Workspace-level Spark behavior

**Remember:**

> **Workspace Spark settings → Control how Spark workloads run in that workspace.**

---

## 1.2 Domain Workspace Settings

Domains organize Fabric workspaces around business areas.

Examples:

```text
Fabric
├── Sales Domain
│   ├── Sales Workspace
│   └── CRM Workspace
│
└── Finance Domain
    ├── Finance Workspace
    └── Reporting Workspace
```

**Remember:**

> **Domain → Organize workspaces by business area and support delegated governance.**

---

## 1.3 OneLake Workspace Settings

OneLake-related workspace settings control aspects of how workspace data participates in OneLake.

Important idea:

> **Workspace settings can affect how data is exposed and managed within the OneLake ecosystem.**

---

## 1.4 Apache Airflow Workspace Settings

Fabric supports Apache Airflow for workflow orchestration.

### What is Airflow?

Apache Airflow is an orchestration platform where workflows are defined as **DAGs (Directed Acyclic Graphs)**.

```text
Task A
  ↓
Task B
  ↓
Task C
```

### Important concepts

- DAG
- Tasks
- Dependencies
- Scheduling
- Workflow orchestration

**Remember:**

> **Airflow → DAG-based workflow orchestration.**

---

# 2. Database Projects

## What is a Database Project?

A database project represents database schema/code as source-controlled development artifacts.

It can help manage:

- Tables
- Views
- Stored procedures
- Database objects
- Schema changes

### Why use it?

Instead of manually changing production databases:

```text
Developer
   ↓
Database Project
   ↓
Git
   ↓
Review
   ↓
Deployment
```

This supports:

- Version control
- Collaboration
- Repeatable deployments
- Lifecycle management

**Remember:**

> **Database Project → Treat database schema/code as deployable source-controlled artifacts.**

---

# 3. Fabric APIs for Lifecycle Management

Fabric APIs can be used to automate administration and lifecycle operations.

Possible uses include:

- Automating deployments
- Managing workspaces
- Managing Fabric items
- Integrating CI/CD into automation
- Performing repetitive administrative operations

**Remember:**

> **Fabric APIs → Automate Fabric lifecycle and administration.**

---

# 4. Full vs Incremental Data Loads

This is a major DP-700 concept.

## 4.1 Full Load

A full load processes the entire source dataset.

```text
Source
  ↓
Read ALL records
  ↓
Destination
```

### Advantages
- Simple
- Easy to understand
- Useful for small datasets

### Disadvantages
- More data movement
- More processing
- Can be expensive for large datasets

---

## 4.2 Incremental Load

An incremental load processes only new or changed data.

```text
Source
  ↓
New / Changed Records
  ↓
Destination
```

Example:

```text
Yesterday: 1,000,000 records
Today:       10,000 new/changed records

Incremental load → process 10,000
```

### Advantages

- Faster
- Less data movement
- Lower processing cost
- Better for large datasets

---

## 4.3 Watermark

A watermark records the point up to which data has already been processed.

Example:

```text
LastProcessedTimestamp = 2026-08-30 10:00
```

Next run:

```text
WHERE ModifiedDate > LastProcessedTimestamp
```

**Remember:**

> **Watermark → Remember where the previous load stopped.**

---

## 4.4 Incremental Load Patterns

Common approaches:

- Timestamp-based
- Increasing ID/key
- CDC
- Change tracking
- Partition-based loading

---

# 5. Change Data Capture (CDC)

## What is CDC?

**Change Data Capture** identifies changes made to source data.

Changes can include:

- Insert
- Update
- Delete

```text
Source Database
      ↓
     CDC
      ↓
┌─────┼─────┐
Insert Update Delete
      ↓
   Fabric
```

### Why use CDC?

Instead of repeatedly reading the entire source:

```text
Full Load
→ Read everything
```

CDC allows:

```text
CDC
→ Read only changes
```

### CDC is useful when:

- Source is large
- Changes are frequent
- Near-real-time/incremental processing is needed
- You need to capture updates and deletes

**Remember:**

> **CDC → Capture inserts, updates, and deletes.**

---

# 6. Prepare Data for a Dimensional Model

Dimensional modeling is important for analytics and BI.

## 6.1 Fact Table

A fact table stores measurable business events.

Examples:

- Sales
- Orders
- Transactions
- Payments

Typical columns:

```text
OrderKey
CustomerKey
ProductKey
DateKey
SalesAmount
Quantity
```

**Remember:**

> **Fact → Business event + measurements.**

---

## 6.2 Dimension Table

A dimension describes the business entities involved in facts.

Examples:

- Customer
- Product
- Date
- Store
- Employee

Example:

```text
CustomerKey
CustomerName
City
State
Country
```

**Remember:**

> **Dimension → Describes the business entity.**

---

## 6.3 Star Schema

A common dimensional model is a star schema.

```text
              Date
               |
               |
Customer ── Fact Sales ── Product
               |
               |
             Store
```

### Benefits

- Easy for BI users
- Simple relationships
- Efficient analytical queries
- Easy aggregation
- Works well with semantic models

**Remember:**

> **Star Schema = Fact in center + Dimensions around it.**

---

## 6.4 Business Key vs Surrogate Key

### Business Key

A key that comes from the source/business system.

Example:

```text
CustomerID = C1001
```

### Surrogate Key

A generated key used inside the analytical model.

Example:

```text
CustomerKey = 101
```

**Remember:**

> **Business key → Source identity**

> **Surrogate key → Warehouse/model identity**

---

## 6.5 Slowly Changing Dimensions

A dimension can change over time.

Example:

```text
Customer moves:
Bangalore → Hubballi
```

The model needs to decide whether to overwrite the old value or preserve history.

### Type 1

Overwrite the old value.

```text
Bangalore → Hubballi
```

No history is retained.

### Type 2

Create a new dimension record and preserve history.

```text
Customer 101 → Bangalore
Customer 205 → Hubballi
```

Common additional columns:

- EffectiveFrom
- EffectiveTo
- IsCurrent

**Remember:**

> **Type 1 → Overwrite**

> **Type 2 → Preserve history**

---

# 7. Denormalization

## What is Denormalization?

Denormalization combines related data to reduce the number of joins.

Normalized:

```text
Customer
   ↓
CustomerAddress
   ↓
CustomerRegion
```

Denormalized:

```text
Customer
Name | City | State | Region
```

### Why use it?

- Simplifies analytical queries
- Reduces joins
- Can improve query performance
- Makes data easier for BI users

### Trade-off

- More duplicated data
- More storage
- More complex update logic

**Remember:**

> **Denormalization → Fewer joins, easier analytics.**

---

# 8. Group and Aggregate Data

Aggregation summarizes detailed data.

Examples:

```sql
SUM()
AVG()
COUNT()
MIN()
MAX()
```

Example:

```sql
SELECT
    ProductId,
    SUM(SalesAmount) AS TotalSales
FROM Sales
GROUP BY ProductId;
```

Conceptually:

```text
Millions of transactions
        ↓
     GROUP BY
        ↓
Summary by product
```

**Remember:**

> **Aggregation → Convert detailed data into summaries.**

---

# 9. Handling Duplicate Data

Duplicate records can occur because of:

- Repeated file ingestion
- Retry operations
- Source-system duplicates
- Incorrect joins
- Reprocessing

### Common approaches

- Identify business key
- Use `DISTINCT` where appropriate
- Use `ROW_NUMBER()`
- Deduplicate before loading
- Use `MERGE`
- Design idempotent pipelines

Example:

```sql
ROW_NUMBER() OVER (
    PARTITION BY CustomerId
    ORDER BY ModifiedDate DESC
)
```

**Remember:**

> **Duplicate handling → Identify the correct record and remove/reject unwanted copies.**

---

# 10. Handling Missing Data

Missing data can appear as:

- NULL
- Empty values
- Missing records
- Missing attributes

Possible approaches:

- Keep NULL when it has meaning
- Replace with default values when appropriate
- Filter invalid records
- Impute values where appropriate
- Flag data-quality problems

**Remember:**

> **Missing data → Detect → Validate → Decide how to handle it.**

---

# 11. Late-Arriving Data

Late-arriving data means data arrives after the expected processing window.

Example:

```text
Day 1 → Sales expected
Day 2 → Sales arrives
```

The pipeline must be able to process the late record without corrupting existing results.

Useful techniques:

- Incremental reprocessing
- Watermarks
- MERGE/upsert
- Event timestamps
- Recalculation of affected aggregates
- Maintaining historical data

**Remember:**

> **Late-arriving data → Data arrives after its expected time.**

---

# 12. Choose the Appropriate Data Store

Different workloads need different Fabric storage engines.

| Requirement | Appropriate choice |
|---|---|
| Structured SQL analytics | Warehouse |
| Data engineering + Spark | Lakehouse |
| Files/raw data | Lakehouse |
| Real-time/time-series analytics | Eventhouse |
| Streaming ingestion/routing | Eventstream |
| External data without copying | OneLake Shortcut |
| Database replication | Mirroring |

### Simple memory

> **Structured SQL → Warehouse**

> **Engineering/Spark → Lakehouse**

> **Real-time → Eventhouse**

> **Streaming routing → Eventstream**

> **Reference external data → Shortcut**

> **Replicate database data → Mirroring**

---

# 13. Transformation Tool Selection

This is a very important exam decision area.

| Requirement | Best fit |
|---|---|
| Low-code transformation | Dataflow Gen2 |
| Complex Python/Spark transformation | Notebook |
| SQL-based transformation | T-SQL |
| Real-time/KQL transformation | KQL |
| Large-scale distributed transformation | Spark |
| Orchestration | Pipeline |

### Dataflow Gen2

Use when:

- Low-code transformation
- Power Query
- Business-user friendly
- Multiple visual transformation steps

### Notebook

Use when:

- Complex logic
- PySpark
- Large-scale processing
- Custom code
- Data science/engineering logic

### T-SQL

Use when:

- Data is in a Warehouse
- SQL transformation is natural
- Relational operations are required

### KQL

Use when:

- Working with Eventhouse/KQL data
- Real-time analytics
- Time-series analysis

**Remember:**

> **Dataflow = Low-code**

> **Notebook = Code/Spark**

> **T-SQL = SQL**

> **KQL = Real-time analytics**

---

# 14. OneLake Shortcuts

## What is a Shortcut?

A shortcut provides a reference to data stored elsewhere without requiring a traditional copy into the destination.

```text
External / Other Fabric Data
          ↓
       Shortcut
          ↓
        OneLake
          ↓
      Fabric Workload
```

### Benefits

- Avoid unnecessary copying
- Reduce data duplication
- Access data where it already exists
- Improve data sharing/interoperability

### Common sources

Depending on supported scenarios:

- OneLake
- ADLS Gen2
- Amazon S3
- Other supported storage

---

## 14.1 Internal vs External Shortcuts

### Internal

Reference data from another location within the Fabric/OneLake ecosystem.

### External

Reference data stored in external storage systems.

**Remember:**

> **Shortcut = Reference instead of copy.**

---

# 15. Mirroring

## What is Mirroring?

Fabric Mirroring replicates data from supported source databases into Fabric.

Conceptually:

```text
Source Database
      ↓
   Mirroring
      ↓
 Fabric / OneLake
      ↓
Analytics
```

### Why use it?

- Near-real-time data replication
- Reduce traditional ETL complexity
- Keep analytical copy synchronized with source
- Useful when operational database data needs to be analyzed in Fabric

### Mirroring vs Pipeline

**Pipeline:**

> Move data according to an orchestration schedule/event.

**Mirroring:**

> Continuously replicate supported source database changes.

**Remember:**

> **Pipeline → Move**

> **Mirroring → Replicate**

---

# 16. Streaming Data Loading Pattern

A streaming pipeline continuously processes arriving events.

```text
Streaming Source
      ↓
Eventstream / Spark Streaming
      ↓
Transform
      ↓
Eventhouse / Lakehouse
      ↓
Real-Time Analytics
```

Important characteristics:

- Continuous ingestion
- Low latency
- Event timestamps
- Windowed aggregation
- Fault/recovery considerations
- Monitoring

**Remember:**

> **Streaming → Continuous processing instead of periodic batch processing.**

---

# 17. Choose a Streaming Engine

## Eventstream

Best for:

- Visual/no-code streaming
- Routing events
- Simple in-stream transformations
- Multiple destinations

## Eventhouse / KQL

Best for:

- Real-time analytics
- Time-series data
- KQL queries
- High-volume event analysis

## Spark Structured Streaming

Best for:

- Complex streaming transformations
- PySpark/code-based processing
- Custom streaming logic
- Large-scale distributed processing

### Simple memory

> **Eventstream → Route**

> **Eventhouse/KQL → Analyze**

> **Spark Structured Streaming → Complex processing**

---

# 18. Native Tables vs OneLake Shortcuts in Real-Time Intelligence

In Real-Time Intelligence, you may work with data stored natively or reference external data through shortcuts.

### Native Table

Data is ingested/stored in the Eventhouse environment.

Use when:

- Data is actively ingested
- Real-time querying is central
- Eventhouse should own the data

### Shortcut

References data stored elsewhere.

Use when:

- Data already exists elsewhere
- You want to avoid unnecessary copying
- Cross-workload access is required

**Remember:**

> **Native table → Ingest/store here**

> **Shortcut → Reference data elsewhere**

---

# 19. Query Acceleration for OneLake Shortcuts

Standard shortcuts reference external data.

Query acceleration is designed to improve query performance for supported shortcut scenarios by optimizing how shortcut data is queried.

### Exam decision

Think:

```text
Need direct reference
      ↓
Standard Shortcut

Need improved query performance for supported shortcut data
      ↓
Query Acceleration
```

**Remember:**

> **Standard Shortcut → Reference**

> **Query Acceleration → Improve supported shortcut query performance**

---

# 20. Spark Structured Streaming

## What is it?

Spark Structured Streaming allows Spark to process continuously arriving data using the DataFrame/Spark SQL programming model.

Conceptually:

```text
Streaming Source
      ↓
Read Stream
      ↓
Transform
      ↓
Write Stream
      ↓
Destination
```

### Important concepts

- Streaming DataFrame
- Read stream
- Write stream
- Checkpointing
- Triggers
- Continuous/incremental processing
- Fault recovery

### Checkpointing

Checkpoints maintain processing state/progress so a streaming job can recover.

**Remember:**

> **Checkpoint → Store streaming progress/state for recovery.**

---

# 21. Streaming Windowing

Windowing groups streaming events into time-based periods.

## 21.1 Tumbling Window

Fixed, non-overlapping windows.

```text
10:00–10:05
10:05–10:10
10:10–10:15
```

Each event belongs to one window.

**Remember:**

> **Tumbling = Fixed + Non-overlapping**

---

## 21.2 Hopping Window

Windows overlap.

Example:

```text
10:00–10:10
10:05–10:15
10:10–10:20
```

**Remember:**

> **Hopping = Fixed duration + Overlap**

---

## 21.3 Sliding Window

Continuously evaluates a moving time range.

Conceptually:

```text
──────────────>
   [Window]
      [Window]
         [Window]
```

**Remember:**

> **Sliding = Moving window**

---

## 21.4 Session Window

Groups events based on periods of activity separated by inactivity.

Example:

```text
User events
● ● ●       ● ●
|---session---|  |session|
```

**Remember:**

> **Session = Activity separated by inactivity gaps**

---

# 22. KQL for Streaming/Real-Time Data

Important KQL concepts include:

```text
where
project
extend
summarize
join
bin()
```

Example:

```kql
SensorData
| where Timestamp > ago(30m)
| summarize AvgTemp = avg(Temperature)
    by bin(Timestamp, 5m), SensorId
```

### `bin()`

Groups timestamps into fixed intervals.

```kql
bin(Timestamp, 5m)
```

**Remember:**

> **bin() → Time bucketing**

---

# 23. Monitor Data Ingestion

Monitoring ingestion means checking whether data is arriving correctly.

Look for:

- Successful loads
- Failed loads
- Data freshness
- Data volume
- Processing duration
- Missing files/events
- Source connectivity
- Throughput

**Remember:**

> **Ingestion monitoring → Is data arriving correctly and on time?**

---

# 24. Monitor Data Transformation

Check:

- Transformation status
- Duration
- Errors
- Input/output row counts
- Data quality
- Spark job stages
- Resource consumption

**Remember:**

> **Transformation monitoring → Did the data change correctly and efficiently?**

---

# 25. Monitor Semantic Model Refresh

A semantic model must refresh successfully so reports contain current data.

Monitor:

- Refresh status
- Duration
- Failure reason
- Data source connectivity
- Refresh schedule
- Dependencies

```text
Warehouse/Lakehouse
       ↓
Semantic Model Refresh
       ↓
Power BI Report
```

**Remember:**

> **Semantic refresh → Ensures BI data is current.**

---

# 26. Configure Alerts

Alerts can notify users when important conditions occur.

Examples:

- Pipeline failure
- Capacity issue
- Data threshold
- Streaming condition
- Refresh failure

Possible notification mechanisms depend on the Fabric feature/integration.

**Remember:**

> **Alert → Detect condition → Notify/Act**

---

# 27. Troubleshoot Pipeline Errors

Common causes:

- Invalid source connection
- Authentication problem
- Incorrect path
- Schema mismatch
- Invalid parameter
- Destination unavailable
- Data type problem
- Timeout
- Permission issue

Troubleshooting flow:

```text
Pipeline Failed
     ↓
Identify failed activity
     ↓
Read error details
     ↓
Check source/destination
     ↓
Check parameters
     ↓
Fix
     ↓
Retry
```

---

# 28. Troubleshoot Dataflow Gen2 Errors

Common causes:

- Source unavailable
- Authentication
- Invalid Power Query transformation
- Data type mismatch
- Schema changes
- Destination problem
- Resource/performance issues

Check:

- Power Query steps
- Error rows
- Source schema
- Destination
- Refresh history

**Remember:**

> **Dataflow troubleshooting → Source → Transformation → Destination**

---

# 29. Troubleshoot Notebook Errors

Common causes:

- Syntax errors
- Missing libraries
- Incorrect paths
- Permission problems
- Schema mismatch
- Spark memory issues
- Shuffle problems
- Data skew
- Cluster/resource issues

Troubleshooting:

```text
Notebook Failure
      ↓
Read error/log
      ↓
Identify failing cell/stage
      ↓
Check data/schema/resources
      ↓
Fix
      ↓
Rerun
```

---

# 30. Troubleshoot Eventstream Errors

Possible causes:

- Source connection failure
- Authentication
- Invalid schema
- Destination unavailable
- Transformation issue
- Event format problem
- Throughput/performance problem

Check:

- Source status
- Eventstream configuration
- Transformation
- Destination
- Event preview/monitoring

---

# 31. Troubleshoot Eventhouse Errors

Possible causes:

- Ingestion failure
- Invalid schema
- KQL syntax error
- Insufficient permissions
- Incorrect table configuration
- Retention/ingestion issues

Check:

- Ingestion status
- KQL query
- Table/schema
- Permissions
- Policies

---

# 32. Troubleshoot T-SQL Errors

Common causes:

- Syntax errors
- Invalid object/table
- Permission issues
- Data type mismatch
- Constraint issues
- Unsupported SQL features
- Incorrect database/schema/table reference

Troubleshooting:

```text
T-SQL Error
   ↓
Read error
   ↓
Check syntax
   ↓
Check object/schema
   ↓
Check permissions
   ↓
Check data types
   ↓
Retry
```

---

# 33. Troubleshoot OneLake Shortcut Errors

Common causes:

- Invalid source path
- Authentication/permission issue
- Source unavailable
- Unsupported source/configuration
- Network/connectivity issue
- Incorrect shortcut configuration

Check:

- Source
- Path
- Permissions
- Shortcut configuration
- Source availability

**Remember:**

> **Shortcut troubleshooting → Source + Path + Permissions + Configuration**

---

# 34. Optimize a Lakehouse Table

Important optimization concepts:

## Small Files

Many tiny files can increase overhead.

```text
Bad:
1 MB
2 MB
1 MB
3 MB
2 MB
...
```

Compaction combines files into more efficient larger files.

### OPTIMIZE

Used to compact Delta table files.

**Remember:**

> **OPTIMIZE → Compact small files.**

### VACUUM

Removes old, unreferenced files according to retention rules.

**Remember:**

> **VACUUM → Clean old unreferenced files.**

### Partitioning

Partitioning can organize data by a useful column such as date.

But excessive partitioning can create too many small files.

**Remember:**

> **Good partitioning → Helps pruning**

> **Too many partitions → Can create small-file problems**

---

# 35. Optimize a Pipeline

Pipeline optimization includes:

- Reduce unnecessary activities
- Use parallel execution when safe
- Avoid unnecessary data movement
- Use parameters
- Reuse pipeline logic
- Use appropriate retry policies
- Avoid serial execution when tasks are independent
- Process data incrementally when possible

Example:

```text
Bad:
A → B → C → D

If independent:
A ─┐
B ─┼→ D
C ─┘
```

**Remember:**

> **Pipeline optimization → Less unnecessary work + appropriate parallelism.**

---

# 36. Optimize a Data Warehouse

Important areas:

- Query optimization
- Reduce unnecessary columns
- Filter appropriately
- Avoid unnecessary joins
- Use appropriate data types
- Review execution/query information
- Use appropriate table design
- Use aggregation/materialization where appropriate
- Monitor capacity/resource usage

**Remember:**

> **Warehouse optimization → Efficient query + good data design + capacity awareness.**

---

# 37. Optimize Eventstreams

Optimization ideas:

- Filter unnecessary events early
- Avoid unnecessary transformations
- Reduce unnecessary destinations
- Aggregate where appropriate
- Monitor throughput
- Design sources/destinations efficiently

**Remember:**

> **Eventstream optimization → Process only the events you need.**

---

# 38. Optimize Eventhouses

Important techniques:

- Appropriate ingestion design
- Efficient KQL queries
- Filter early
- Project only required columns
- Use `summarize` appropriately
- Use `bin()` for time-based aggregation
- Materialized views for repeated aggregation patterns
- Appropriate retention policies
- Monitor ingestion/query performance

**Remember:**

> **Eventhouse optimization → Efficient ingestion + efficient KQL + appropriate data lifecycle.**

---

# 39. Optimize Spark Performance

This is a major topic.

## 39.1 Partitioning

Partition data so work can be distributed efficiently.

## 39.2 Repartition

Changes the number/distribution of partitions and can cause a shuffle.

## 39.3 Coalesce

Can reduce partitions with less shuffle in suitable situations.

**Remember:**

> **Repartition → Redistribute**

> **Coalesce → Reduce partitions**

---

## 39.4 Shuffle

Shuffle moves data between executors/partitions.

Large shuffles can be expensive.

Common causes:

- Large joins
- GroupBy
- Repartition
- Sorting

**Remember:**

> **Shuffle → Expensive data movement between partitions.**

---

## 39.5 Broadcast Join

Broadcast a small dataset to executors so a large shuffle can sometimes be avoided.

Use when one side of the join is sufficiently small.

**Remember:**

> **Small table + huge table → Consider broadcast join.**

---

## 39.6 Data Skew

Data skew occurs when some partitions contain much more data than others.

Example:

```text
Partition 1 → 1 GB
Partition 2 → 1 GB
Partition 3 → 1 GB
Partition 4 → 100 GB
```

One task becomes a bottleneck.

**Remember:**

> **Data skew → Uneven data distribution.**

---

## 39.7 Caching

Caching can keep reused data in memory/storage for repeated operations.

Use when the same dataset is reused multiple times.

**Remember:**

> **Cache → Useful for repeated access.**

Do not cache everything because it consumes resources.

---

## 39.8 Filter Early

Filtering data before expensive operations can reduce processing.

```text
Large Dataset
     ↓
Filter
     ↓
Smaller Dataset
     ↓
Join / Aggregate
```

**Remember:**

> **Filter early → Process less data.**

---

# 40. Optimize Query Performance

## General principles

- Select only required columns.
- Filter early.
- Avoid unnecessary joins.
- Use appropriate predicates.
- Aggregate appropriately.
- Avoid repeatedly scanning large datasets.
- Use appropriate indexes/optimization features where supported.
- Review query execution information.
- Use materialization where supported and beneficial.

### Lakehouse/Spark

Consider:

- File size
- Partitioning
- Predicate pushdown
- Data skipping
- Small-file management

### Warehouse

Consider:

- Query plan
- Statistics
- Data layout
- Join strategy
- Aggregation

### Eventhouse

Consider:

- KQL filtering
- Projection
- `summarize`
- `bin()`
- Materialized views

---

# 41. Data Freshness

Data freshness measures how current the available data is.

Example:

```text
Source updated → 10:00
Fabric data updated → 10:05
Freshness delay → 5 minutes
```

Monitor:

- Last successful load
- Last refresh
- Event arrival time
- Pipeline duration
- Processing latency

**Remember:**

> **Data freshness → How up-to-date is the data?**

---

# 42. Data Volume Monitoring

Monitor:

- Number of records
- File count
- File size
- Event volume
- Growth rate

Unexpected volume can indicate:

- Duplicate loads
- Missing data
- Source changes
- Pipeline errors

**Remember:**

> **Volume monitoring → Is the amount of data expected?**

---

# 43. Data Quality Monitoring

Important checks:

- Nulls
- Duplicates
- Invalid data types
- Invalid ranges
- Missing records
- Referential integrity where applicable
- Unexpected row counts

**Remember:**

> **Data quality → Is the data correct and usable?**

---

# 44. Decision Matrix – DP-700

| Scenario | Choose |
|---|---|
| Low-code ETL | Dataflow Gen2 |
| Complex Spark processing | Notebook |
| SQL transformation | T-SQL |
| Real-time query/transform | KQL |
| Orchestration | Pipeline |
| External data without copying | Shortcut |
| Database replication | Mirroring |
| Streaming routing | Eventstream |
| Real-time analytics | Eventhouse |
| Complex streaming code | Spark Structured Streaming |
| Structured SQL analytics | Warehouse |
| Big-data engineering | Lakehouse |
| Source changes only | CDC |
| Complete reload | Full load |
| New/changed records only | Incremental load |

---

# 45. Final DP-700 Missing-Concept Checklist

## Implement and Manage

- [x] Workspace settings
- [x] Spark workspace settings
- [x] Domain settings
- [x] OneLake settings
- [x] Apache Airflow basics
- [x] Git/version control
- [x] Database projects
- [x] Deployment pipelines
- [x] Fabric APIs
- [x] Workspace security
- [x] Item security
- [x] Object security
- [x] Folder/file security
- [x] RLS
- [x] CLS
- [x] DDM
- [x] Sensitivity labels
- [x] Endorsement
- [x] Audit logs
- [x] OneLake security
- [x] Pipeline orchestration
- [x] Parameters/dynamic expressions
- [x] Scheduling
- [x] Event-based triggers

## Ingest and Transform

- [x] Full loads
- [x] Incremental loads
- [x] Watermarks
- [x] CDC
- [x] Dimensional modeling
- [x] Fact tables
- [x] Dimension tables
- [x] Star schema
- [x] Surrogate keys
- [x] Slowly changing dimensions
- [x] Denormalization
- [x] Aggregation
- [x] Duplicate handling
- [x] Missing data
- [x] Late-arriving data
- [x] Data-store selection
- [x] Transformation-tool selection
- [x] OneLake shortcuts
- [x] Internal/external shortcuts
- [x] Mirroring
- [x] Streaming loading patterns
- [x] Streaming-engine selection
- [x] Native tables vs shortcuts
- [x] Query acceleration
- [x] Eventstreams
- [x] Spark Structured Streaming
- [x] KQL
- [x] Windowing
- [x] Tumbling windows
- [x] Hopping windows
- [x] Sliding windows
- [x] Session windows

## Monitor and Optimize

- [x] Data ingestion monitoring
- [x] Data transformation monitoring
- [x] Semantic model refresh monitoring
- [x] Alerts
- [x] Pipeline troubleshooting
- [x] Dataflow troubleshooting
- [x] Notebook troubleshooting
- [x] Eventhouse troubleshooting
- [x] Eventstream troubleshooting
- [x] T-SQL troubleshooting
- [x] Shortcut troubleshooting
- [x] Lakehouse optimization
- [x] OPTIMIZE
- [x] VACUUM
- [x] Pipeline optimization
- [x] Warehouse optimization
- [x] Eventstream optimization
- [x] Eventhouse optimization
- [x] Spark optimization
- [x] Query optimization
- [x] Data freshness
- [x] Data volume
- [x] Data quality

---

# 🧠 Ultimate DP-700 Memory Map

```text
                         DP-700
                           |
        ┌──────────────────┼──────────────────┐
        ↓                  ↓                  ↓
 IMPLEMENT/MANAGE      INGEST/TRANSFORM   MONITOR/OPTIMIZE
        ↓                  ↓                  ↓
 Workspace             Full/Incremental   Monitoring
 Git                   CDC                Troubleshooting
 Deployment            Dimensional Model  Lakehouse
 Security              Shortcuts          Warehouse
 Governance             Mirroring          Spark
 Orchestration          Batch/Streaming    Eventhouse
 Admin                  KQL                Query Performance
        ↓                  ↓                  ↓
        └──────────────────┼──────────────────┘
                           ↓
              Fabric Data Engineering
```

# 🔑 Final One-Line Memories

> **Git → Version control**

> **Deployment Pipeline → Promote**

> **Airflow → DAG orchestration**

> **Full Load → Everything**

> **Incremental Load → New/changed data**

> **CDC → Inserts + Updates + Deletes**

> **Fact → Measures/events**

> **Dimension → Describes entities**

> **Star Schema → Fact + Dimensions**

> **Shortcut → Reference, don't copy**

> **Mirroring → Replicate**

> **Eventstream → Route streaming data**

> **Eventhouse → Analyze real-time data**

> **Spark Structured Streaming → Code-based streaming**

> **Tumbling → Non-overlapping windows**

> **Hopping → Overlapping windows**

> **Session → Activity-based windows**

> **RLS → Rows**

> **CLS → Columns**

> **DDM → Mask values**

> **Lineage → Data dependencies**

> **Monitoring Hub → Job/run monitoring**

> **Capacity Metrics → Capacity health**

> **OPTIMIZE → Compact files**

> **VACUUM → Remove old unreferenced files**

> **Shuffle → Data movement**

> **Broadcast → Send small table to executors**

> **Data skew → Uneven partitions**

> **Dataflow → Low-code**

> **Notebook → Spark/code**

> **T-SQL → SQL transformation**

> **KQL → Real-time analytics**

> **Pipeline → Orchestrate**

---



