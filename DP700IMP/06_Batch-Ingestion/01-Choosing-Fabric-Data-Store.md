# DP-700 — Choosing a Fabric Data Store: Explained Notes

## 1. Core Decision Rule

Choose a Fabric data store by evaluating these factors in this order:

1. **Language surface and DML requirement** — Spark, KQL, read-only T-SQL, or full T-SQL DML.
2. **Workload type** — data engineering, SQL analytics, streaming/time-series, or transactional/OLTP.
3. **Latency requirement** — seconds/sub-seconds, near-real-time, or a less time-sensitive workload.
4. **Primary consumer** — data engineers, SQL analysts, streaming analysts, or operational applications.

**Memory trick:** Language → Workload → Latency → Consumer.

## 2. The Four Data Stores

### A. Lakehouse

**Primary fit:** Spark-first data engineering with occasional SQL consumption.

Choose a lakehouse when:
- The team primarily uses Spark or PySpark.
- Data engineering involves processing and transforming data.
- SQL is used occasionally for analytics.
- A read-only SQL analytics endpoint meets the SQL requirement.

**SQL capability:** The lakehouse SQL analytics endpoint is read-only. It does not provide full T-SQL DML through that endpoint.

**Example:** A data engineer transforms raw data into curated Delta tables with PySpark; analysts query the tables using SQL.

### B. Warehouse

**Primary fit:** SQL-first data engineering and analytics.

Choose a warehouse when:
- The entire team, including the data-loading process, is SQL-only.
- Full T-SQL DML is required.
- The workload is primarily SQL-based warehousing and analytics.

**SQL capability:** The warehouse's primary SQL surface supports full T-SQL DML.

**Example:** A team loads and manages analytical tables using SQL scripts and queries.

### C. Eventhouse

**Primary fit:** Streaming, telemetry, time-series, and KQL-native analytics.

Choose Eventhouse when:
- The scenario mentions KQL.
- Data arrives continuously from devices, applications, or telemetry sources.
- The workload is time-series analytics.
- Query freshness or latency is measured in seconds, sub-seconds, or near-real-time.

**SQL capability:** Eventhouse has a read-only SQL analytics endpoint; KQL is its native analytics language.

**Example:** A company analyzes device telemetry and detects unusual readings within seconds.

### D. Fabric SQL database

**Primary fit:** OLTP and operational applications that also need analytics availability.

Choose Fabric SQL database when:
- The workload requires OLTP characteristics such as foreign keys, high concurrency, and ACID transactions.
- Full T-SQL DML is required.
- Analytics on the same data should be available through automatic, pipeline-free OneLake mirroring.

**SQL capability:** The primary SQL surface supports full T-SQL DML.

**Example:** An application stores customer orders in a transactional database, while analysts need access to the data for analytics through OneLake mirroring.

**Exam reminder:** Do not rule it out merely because a scenario calls it an application or operational database.

## 3. SQL Surface: Read-Only vs Full T-SQL DML

| SQL surface | Data stores |
|---|---|
| **Read-only T-SQL SQL analytics endpoint** | Lakehouse, Eventhouse, mirrored database |
| **Full T-SQL DML on primary surface** | Warehouse, Fabric SQL database |

**DML** means Data Manipulation Language. Common examples include `INSERT`, `UPDATE`, and `DELETE`.

Exam clue:
- If the scenario says **read-only T-SQL**, consider a lakehouse, Eventhouse, or mirrored database SQL analytics endpoint.
- If the scenario requires **full T-SQL DML**, consider a warehouse or Fabric SQL database.

## 4. Workload and Latency Clues

| Scenario clue | Likely store |
|---|---|
| Spark/PySpark-first data engineering; occasional SQL | Lakehouse |
| Entire team and loading process are SQL-only | Warehouse |
| KQL, telemetry, time-series | Eventhouse |
| Seconds, sub-second, near-real-time streaming analytics | Eventhouse |
| OLTP, foreign keys, high concurrency, ACID | Fabric SQL database |
| Operational database with pipeline-free analytics via automatic OneLake mirroring | Fabric SQL database |
| Full T-SQL DML | Warehouse or Fabric SQL database |
| Read-only SQL analytics endpoint | Lakehouse, Eventhouse, or mirrored database |

## 5. OneLake and Open Table Format

All four stores land data in OneLake in open table format, according to the supplied study material.

**Exam trap:** Do not use “the data lands in OneLake anyway” as the deciding factor. OneLake availability does not distinguish these stores. Focus on language, DML, workload, latency, and consumer.

## 6. Decision Flow

1. **Does the scenario require OLTP features** such as foreign keys, high concurrency, and ACID transactions?
   - Yes → Consider **Fabric SQL database**.
2. **Is the workload streaming, telemetry, or time-series, with KQL or seconds-level freshness?**
   - Yes → Consider **Eventhouse**.
3. **Is the team SQL-only, including data loading, and does it need full T-SQL DML?**
   - Yes → Consider **Warehouse**.
4. **Is the workload Spark-first, with occasional SQL consumption?**
   - Yes → Consider **Lakehouse**.
5. If the scenario is about a mirrored database, check whether the requirement is a read-only SQL analytics endpoint or full DML on the primary store.

## 7. Exam Tips

- Match the **language surface and DML requirement first**, then confirm workload and latency.
- **Spark-first + occasional SQL** → Lakehouse.
- **SQL-only team, including loading** → Warehouse.
- **Seconds-level freshness, KQL, telemetry, or time-series** → Eventhouse.
- **OLTP + foreign keys + high concurrency + ACID + pipeline-free analytics** → Fabric SQL database.
- **Read-only T-SQL** → Lakehouse, Eventhouse, or mirrored database SQL analytics endpoint.
- **Full T-SQL DML** → Warehouse or Fabric SQL database primary surface.
- All four stores use OneLake/open table format in the supplied material; that fact alone does not decide the answer.

## 8. Key Takeaways

- Store selection depends on language surface, DML capability, workload, latency, and primary consumer—not simply where data lands.
- Lakehouse and Eventhouse expose read-only SQL analytics endpoints.
- Warehouse and Fabric SQL database support full T-SQL DML on their primary SQL surfaces.
- Eventhouse is purpose-built for streaming, time-series, KQL-native workloads with near-real-time query latency.
- Fabric SQL database is Fabric's OLTP option, with automatic pipeline-free near-real-time mirroring into OneLake.

## 9. One-Minute Revision

- **Spark-first?** Lakehouse.
- **SQL-only team and loading?** Warehouse.
- **KQL, telemetry, time-series, seconds-level freshness?** Eventhouse.
- **OLTP, foreign keys, ACID, high concurrency?** Fabric SQL database.
- **Read-only SQL endpoint?** Lakehouse / Eventhouse / mirrored database.
- **Full T-SQL DML?** Warehouse / Fabric SQL database.
- **All in OneLake?** Not a deciding factor.
