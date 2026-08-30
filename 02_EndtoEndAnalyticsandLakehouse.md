# Lesson 2 – End-to-End Analytics and the Lakehouse

## 2.1 – Introduction to End-to-End Analytics Using Microsoft Fabric

### 1. Microsoft Fabric
- Unified SaaS analytics platform.
- Brings data engineering, integration, warehousing, real-time analytics, data science, and BI together.
- Supports the complete analytics lifecycle.

**Remember:** `Fabric = One platform for end-to-end analytics`

### 2. OneLake
- Unified logical data lake for an organization.
- Central place for Fabric workloads to access organizational data.
- Supports open formats such as Delta Lake and Parquet.

**Analogy:** `OneDrive → files | OneLake → organizational data`

### 3. Core Workloads
- **Data Factory** → ingestion and orchestration
- **Data Engineering** → Spark/lakehouse engineering
- **Data Warehouse** → SQL analytics
- **Real-Time Intelligence** → real-time/event analytics
- **Data Science** → ML and data science
- **Power BI** → reporting and visualization

### 4. Shared Data
- Different Fabric workloads can work with data in OneLake.
- Reduces unnecessary movement and duplication.

### 5. End-to-End Analytics
`Data Sources → Ingest → Store → Transform → Analyze → Visualize → Business Insights`

### 6. Data Ingestion
- Data Factory provides ingestion and orchestration capabilities.

### 7. Data Storage
- OneLake provides the unified storage layer.
- Lakehouse organizes and works with data stored in OneLake.

### 8. Data Transformation
- **Dataflow Gen2** → low-code transformations
- **Spark** → code-based/large-scale transformations
- **SQL** → SQL-based transformations

### 9. Data Analysis
- Lakehouse/Spark
- Warehouse
- SQL
- KQL
- Semantic models

### 10. Power BI
- Used for business intelligence and visualization.
- Creates reports, dashboards, and visualizations.

**Remember:** `Power BI = Business Intelligence + Visualization`

### 11. Real-Time Analytics
`Real-Time Sources → Eventstream → Eventhouse → KQL → Dashboard / Activator`

### 12. Workspaces
- A workspace is a container for Fabric items.
- Can contain Lakehouses, Warehouses, Notebooks, Pipelines, Dataflows, Eventhouses, Semantic Models, and Reports.

**Remember:** `Workspace = Container for Fabric items`

### 13. Capacity
- Provides compute resources used to run Fabric workloads.
- Associated with a SKU that determines available resources.

**Remember:** `Workspace = Where items are organized | Capacity = Compute resources`

### 14. Security and Governance
- Fabric provides access control, permissions, security, and governance capabilities.

---

## 2.2 – Get Started with Lakehouse in Microsoft Fabric

### 1. What is a Lakehouse?
- Combines data-lake flexibility with structured analytics capabilities.
- Built on OneLake.
- Lakehouse tables use Delta Lake.

**Remember:** `Lakehouse = Files + Tables + Spark + SQL analytics`

### 2. Files
- Stores file-based data.
- Can contain CSV, JSON, Parquet, and other supported files.
- Suitable for raw, unstructured, or semi-structured data.

**Remember:** `Files → File-based/raw data`

### 3. Tables
- Contains structured data.
- Lakehouse tables use Delta format.
- Can be queried using Spark and the SQL analytics endpoint.

**Remember:** `Tables → Structured Delta data`

### 4. Loading Data
- Manual upload
- Dataflow Gen2
- Data Pipeline / Copy activity
- Spark notebooks
- Shortcuts

### 5. SQL Analytics Endpoint
- Provides T-SQL querying of Lakehouse tables.
- Primarily used for read-only SQL analytics.

**Remember:** `Lakehouse → Spark for engineering + SQL endpoint for SQL analytics`

### 6. Default Semantic Model
- A Lakehouse can provide a default semantic model.
- Allows Lakehouse data to be consumed by Power BI.

`Lakehouse → Semantic Model → Power BI`

### 7. Shortcuts
- Reference data stored elsewhere without physically copying it into the Lakehouse.
- Examples include supported Fabric locations, ADLS Gen2, and Amazon S3.

**Remember:** `Shortcut = Reference data without copying`

### 8. Lakehouse + Spark
- Spark is tightly integrated with Lakehouse.
- Can read, transform, create tables, and write data.

### 9. Lakehouse + SQL
- Lakehouse tables can be accessed through SQL analytics.

`Lakehouse → Delta Tables → Spark / SQL Endpoint → Power BI`

### 10. Lakehouse vs Warehouse

| Lakehouse | Warehouse |
|---|---|
| Files + tables | Primarily structured tables |
| Strong Spark integration | Strong T-SQL integration |
| Data engineering + analytics | SQL analytics |
| Supports file-based data | Primarily structured analytical data |
| Delta tables | Warehouse-managed storage |

**Remember:** `Lakehouse → Spark + Files + Delta | Warehouse → T-SQL + Structured Analytics`

### 11. Typical Workflow
`Create Lakehouse → Load Raw Data → Files → Transform → Tables → SQL / Spark → Power BI`

---

## 2.3 – Use Apache Spark in Microsoft Fabric

> Spark fundamentals were covered in Lesson 1.3. This section focuses on Spark + Lakehouse integration.

### 1. Spark + Lakehouse Integration
- Spark is tightly integrated with Fabric Lakehouse.
- Spark can directly work with Lakehouse files and tables.

**Remember:** `Lakehouse + Spark = Data Engineering`

### 2. Notebook + Lakehouse
- A notebook can be associated with a Lakehouse.
- This makes it convenient to access Lakehouse data from Spark.

### 3. Lakehouse Data Access
Spark can work directly with:
- Files
- Delta tables

### 4. Spark SQL + Lakehouse
- Spark SQL can query Lakehouse tables.
- Lakehouse tables use Delta format.

### 5. Managed Tables
- Managed tables have their data and metadata managed by the Lakehouse.
- Stored in the Lakehouse Tables area.
- Use Delta format.

**Remember:** `Managed table → Managed by Lakehouse`

### 6. Files vs Tables
`Lakehouse → Files (Raw/File Data)`  
`Lakehouse → Tables (Delta Data)`

### 7. Raw Data → Structured Table
`Raw Files → Spark → Clean / Transform → Delta Table`

### 8. Medallion Architecture
`Bronze → Silver → Gold`

- Bronze → Raw
- Silver → Cleaned
- Gold → Business-ready

**Remember:** `Lakehouse stores → Spark processes → Delta tables store structured results`

---

## 2.4 – Work with Delta Lake Tables

### 1. What is Delta Lake?
- Open-source storage layer/table format built on Parquet.
- Adds database-like capabilities to file-based data.
- Fabric Lakehouse tables use Delta Lake.

**Remember:** `Delta = Parquet + Transactions + Versioning + Schema Management`

### 2. Delta Lake and Parquet
- Parquet stores actual data in columnar files.
- Delta adds transaction and table-management capabilities.
- Delta maintains a transaction log.

```text
Delta Table
   ├── Parquet Files
   └── _delta_log
```

### 3. ACID Transactions
- **Atomicity**
- **Consistency**
- **Isolation**
- **Durability**

Provides reliable concurrent reads and writes.

### 4. Transaction Log – `_delta_log`
- Stores transaction/history information.
- Records changes to the Delta table.
- Enables version history, time travel, and reliable table operations.

**Remember:** `_delta_log = Transaction + History Information`

### 5. Time Travel
- Access previous versions of a Delta table.
- Useful for auditing, debugging, historical comparison, and recovery.

### 6. Restore / Rollback
- Previous table states can be restored using supported Delta operations.
- Useful after incorrect changes.

### 7. Schema Enforcement
- Delta can enforce the expected table schema during writes.
- Invalid or incompatible schema changes can be rejected.

**Remember:** `Schema enforcement = Protect the schema`

### 8. Schema Evolution
- Allows intentional schema changes where supported/configured.
- Example: adding a new column.

**Remember:** `Enforcement → Prevent unwanted changes | Evolution → Allow intentional changes`

### 9. UPDATE
- Modifies existing records.

`UPDATE → Modify existing data`

### 10. DELETE
- Removes records matching a condition.

`DELETE → Remove data`

### 11. MERGE
- Combines incoming data with existing table data.
- Commonly used for upsert operations.

`Upsert = UPDATE + INSERT`

### 12. MERGE for Incremental Loads
- Useful when only new or changed records need to be applied.
- Avoids rebuilding the entire target table.

### 13. OPTIMIZE
- Compacts small files into larger files.
- Helps improve table performance.

`OPTIMIZE = Compact files`

### 14. VACUUM
- Removes old, unreferenced files according to retention rules.
- Helps reduce unnecessary storage.
- Can affect availability of older versions for time travel.

`VACUUM = Clean old/unreferenced files`

### 15. Delta in Lakehouse
```text
Lakehouse
    ↓
Delta Table
   ↙       ↘
Parquet   _delta_log
Files        ↓
           Transactions
```

### 16. Why Delta Lake Matters
- ACID transactions
- Transaction history
- Time travel
- Schema enforcement
- Schema evolution
- UPDATE
- DELETE
- MERGE
- Optimization capabilities

**Remember:** `Delta turns file-based data into a reliable table format for analytics.`

---

## 2.5 – Ingest Data with Dataflow Gen2 — Recap

### 1. Purpose
- Ingest, transform, and load data.
- Based on Power Query.

`Dataflow Gen2 = Low-code ETL`

### 2. Basic Flow
`Connect → Transform → Load`

### 3. Lakehouse Integration
`Source → Dataflow Gen2 → Transform → Lakehouse Tables`

### 4. When to Prefer Dataflow Gen2
- Low-code/visual transformations.
- Straightforward ETL using Power Query.

### 5. Dataflow Gen2 vs Spark
- **Dataflow Gen2 → Low-code / Power Query**
- **Spark → Code-based / Complex processing**

**Remember:** `Dataflow Gen2 = Power Query + Low-code transformation + Load`

---

## 2.6 – Orchestrate Processes and Data Movement — Recap

### 1. Purpose
- Data Pipelines orchestrate and automate workflows.
- They coordinate different activities.

`Pipeline = Orchestration`

### 2. Pipeline Activities
- **Copy Data** → Move data
- **Dataflow** → Run Dataflow Gen2
- **Notebook** → Run Spark notebook
- **Stored Procedure** → Execute SQL logic where supported

### 3. Control Flow
- If Condition
- For Each
- Until
- Wait
- Set Variable

### 4. Triggers
- **On demand** → Manual
- **Scheduled** → Schedule-based
- **Event-based** → Triggered by an event

### 5. Parameters
- Make pipelines reusable.
- Same pipeline can process different files, dates, paths, or inputs.

### 6. Dependencies
- Activities can depend on previous activities.

`Copy Data → Success → Dataflow Gen2 → Success → Notebook`

### 7. Monitoring
- Status
- Duration
- Activity execution
- Errors
- Failures

**Remember:** `Pipeline = Trigger + Activities + Dependencies + Control Flow + Monitoring`

---

## 2.7 – Organize Medallion Architecture

### 1. What is Medallion Architecture?
- A data design pattern.
- Organizes data into progressively refined layers.
- Commonly used in Lakehouse architectures.

**Remember:** `Bronze → Raw → Silver → Clean → Gold → Business-ready`

### 2. Bronze Layer – Raw
- Contains raw or minimally processed source data.
- Kept close to how it was received.
- Useful for traceability, auditing, reprocessing, and recovery.

**Remember:** `Bronze = Preserve raw data`

### 3. Silver Layer – Cleaned/Validated
- Contains cleaned and validated data.
- Common operations:
  - Remove duplicates
  - Handle missing values
  - Correct data types
  - Standardize formats
  - Apply data-quality rules
  - Conform schemas

**Remember:** `Silver = Clean + Validate + Standardize`

### 4. Gold Layer – Curated
- Contains business-ready data.
- May be aggregated, joined, modeled, and curated for reporting.

**Remember:** `Gold = Business-ready`

### 5. Why Use Medallion Architecture?
- **Traceability** → Bronze preserves source-level data.
- **Separation of concerns** → Each layer has a different responsibility.
- **Reusability** → One Silver dataset can support multiple Gold outputs.
- **Incremental processing** → Layers can be processed incrementally where appropriate.

### 6. Fabric Tools and Medallion Architecture
```text
Source
  ↓
Pipeline / Dataflow
  ↓
Bronze
  ↓
Spark / Dataflow
  ↓
Silver
  ↓
Spark / Dataflow
  ↓
Gold
  ↓
Power BI
```

### 7. Physical Organization
- Medallion Architecture is a **logical design pattern**.
- Layers can be organized using separate Lakehouses, separate areas within a Lakehouse, or other appropriate structures.

### 8. Pipeline Orchestration
`Pipeline → Bronze Ingestion → Silver Transformation → Gold Transformation`

### 9. Not Fabric-Specific
- Medallion Architecture is a general data architecture pattern.
- It is not unique to Microsoft Fabric.

**Remember:** `Bronze → Preserve | Silver → Clean | Gold → Serve the business`

---

# Lesson 2 – Final Revision

| Topic | Main Idea |
|---|---|
| **2.1 Fabric** | Unified SaaS analytics platform |
| **OneLake** | Unified organizational data lake |
| **Workspace** | Container for Fabric items |
| **Capacity** | Compute resources |
| **2.2 Lakehouse** | Files + Tables + Spark + SQL |
| **Files** | Raw/file-based data |
| **Tables** | Delta structured data |
| **SQL Endpoint** | T-SQL analytics |
| **Shortcuts** | Reference without copying |
| **2.3 Spark + Lakehouse** | Spark for Lakehouse engineering |
| **2.4 Delta Lake** | Reliable table format on Parquet |
| **`_delta_log`** | Transaction/history information |
| **MERGE** | Upsert |
| **OPTIMIZE** | Compact files |
| **VACUUM** | Remove old/unreferenced files |
| **2.5 Dataflow Gen2** | Low-code transformation |
| **2.6 Pipeline** | Orchestration |
| **2.7 Medallion** | Bronze → Silver → Gold |

## 🧠 Lesson 2 in One Picture

```text
                 MICROSOFT FABRIC
                        ↓
                     OneLake
                        ↓
                    Lakehouse
                        ↓
              ┌─────────┴─────────┐
              ↓                   ↓
            Files               Tables
              ↓                   ↓
            Bronze              Delta
              ↓                   ↓
             Spark          SQL Endpoint
              ↓                   ↓
            Silver               BI
              ↓
             Gold
              ↓
           Power BI
```

## Final Memory

> **OneLake → Stores**

> **Lakehouse → Organizes**

> **Spark → Processes**

> **Delta → Manages reliable tables**

> **Pipeline → Orchestrates**

> **Dataflow → Low-code transforms**

> **Bronze → Silver → Gold → Business insights**


