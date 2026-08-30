# Lesson 4 – Data Warehouse

## 4.1 – Introduction to End-to-End Analytics — Recap

### 1. Microsoft Fabric
- **Microsoft Fabric** is a unified SaaS analytics platform.
- It combines data engineering, data integration, data warehousing, real-time analytics, data science, and Power BI.

### 2. OneLake
- **OneLake** is the unified data lake for the organization.
- It provides a common data foundation for Fabric workloads.

### 3. Workspaces
- A **workspace** is a container for Fabric items.
- It provides organization, collaboration, and access control.

### 4. Capacity
- **Capacity** provides compute resources for Fabric workloads.
- Multiple Fabric workloads can share the same capacity.

### 5. End-to-End Analytics Flow

```text
Ingest
  ↓
Store
  ↓
Transform
  ↓
Analyze
  ↓
Visualize
  ↓
Act
```

### 6. Lesson 4 Focus
- Lesson 4 focuses on the **Data Warehouse** experience.
- Warehouse is primarily designed for **structured analytical data and T-SQL-based workloads**.

**Remember:**
> **Warehouse → Structured Data + T-SQL + Analytics**

---

## 4.2 – Get Started with Data Warehouse

### 1. What is a Fabric Data Warehouse?
- A **Fabric Data Warehouse** is a fully managed, **T-SQL-first analytical data store**.
- It is designed primarily for **structured relational data**.
- It is suitable for SQL developers, analysts, and BI workloads.

**Remember:**
> **Warehouse = Structured Data + T-SQL + Analytics**

### 2. Warehouse vs Lakehouse

| | **Lakehouse** | **Warehouse** |
|---|---|---|
| Primary interface | Spark / Lakehouse experience | T-SQL |
| Data | Structured + semi/unstructured files | Structured relational data |
| Main users | Data engineers/data scientists | SQL analysts/BI developers |
| SQL | SQL analytics endpoint | T-SQL |
| Write approach | Spark/Dataflow/Pipeline | T-SQL + supported ingestion tools |
| Main strength | Data engineering + big data | SQL analytics + relational modeling |

**Remember:**
> **Lakehouse → Spark + Files + Delta Tables**

> **Warehouse → Structured Data + T-SQL**

### 3. OneLake Integration
- Fabric Warehouse participates in the broader **OneLake ecosystem**.
- Warehouse and Lakehouse are different Fabric experiences.
- Do not treat Warehouse as simply a Lakehouse with a SQL interface.

### 4. T-SQL Support
Fabric Warehouse provides extensive T-SQL support, including:
- Tables
- Views
- Stored procedures
- Functions where supported
- Transactions
- `INSERT`
- `UPDATE`
- `DELETE`
- `MERGE`
- `COPY INTO`

### Foreign Keys
- Foreign keys are **informational** in Fabric Warehouse rather than being enforced in the same way as a traditional relational database.

### 5. Loading Data
Common approaches include:
- T-SQL
- `INSERT`
- `COPY INTO`
- Data Pipelines
- Dataflow Gen2
- Other supported Fabric integration methods

### 6. Cross-Database Querying
- Fabric supports querying across supported Fabric data stores in appropriate scenarios.
- Three-part naming:

```text
Database.Schema.Table
```

**Remember:**
> **Three-part naming = Database.Schema.Table**

### 7. SQL Analytics Endpoint vs Warehouse

#### Lakehouse SQL Analytics Endpoint
- Provides SQL access to Lakehouse tables.
- Primarily **read-only**.

#### Warehouse
- T-SQL-first analytical store.
- Supports supported read/write T-SQL operations.

**Remember:**
> **Lakehouse SQL Endpoint → Query**

> **Warehouse → Query + Write**

### 8. Semantic Model
- Warehouse data can be used with Power BI semantic modeling.

```text
Warehouse
    ↓
Semantic Model
    ↓
Power BI
```

### 9. When to Choose Warehouse
Choose Warehouse when:
- Data is primarily structured.
- Team is SQL/T-SQL focused.
- Relational analytical modeling is required.
- T-SQL write capabilities are needed.
- Workload is primarily analytical/BI.

**Remember:**
> **SQL-first + Structured Data → Warehouse**

---

## 4.3 – Load Data into Microsoft Fabric

### 1. Main Loading Methods

#### A. `COPY INTO`
- T-SQL command for loading data into Fabric Warehouse.
- Useful for **bulk loading files**.
- Common file formats include CSV and Parquet.

```sql
COPY INTO dbo.Sales
FROM '.../sales/*.csv'
WITH (
    FILE_TYPE = 'CSV',
    FIRSTROW = 2
);
```

**Remember:**
> **COPY INTO → Bulk file loading**

#### B. Data Pipelines
- Data Pipelines can use the **Copy Data** activity to load data.
- Useful for:
  - Scheduling
  - Automation
  - Multiple activities
  - Dependencies
  - Monitoring

**Remember:**
> **Pipeline → Move + Orchestrate + Schedule**

#### C. Dataflow Gen2
- Useful when data needs transformation before loading.
- Uses Power Query.
- Suitable for low-code ETL.

```text
Source
  ↓
Dataflow Gen2
  ↓
Transform
  ↓
Warehouse
```

**Remember:**
> **Dataflow Gen2 → Transform + Load**

#### D. Cross-Item T-SQL
- Supported Fabric items can be queried together in appropriate scenarios.
- T-SQL can be used to insert/query data from supported Fabric sources.

### 2. Staging Pattern
- Land incoming data in a staging area/table first.
- Validate or transform the data.
- Load it into final Warehouse tables.

```text
Source Files
     ↓
Staging
     ↓
Validation / Transformation
     ↓
Final Warehouse Table
```

**Remember:**
> **Staging = Land → Validate/Transform → Final**

### 3. File Formats

#### CSV
- Text-based.
- Simple and widely used.

#### Parquet
- Columnar format.
- Efficient for analytical workloads.
- Well suited to large analytical datasets.

**Remember:**
> **CSV → Simple/Text**

> **Parquet → Columnar/Analytics**

### 4. Error Handling
- Loading operations provide supported options for handling errors.
- `MAXERRORS` can control how many errors are tolerated before a supported load fails.

**Remember:**
> **Error handling → Decide when a load should fail**

### 5. Idempotency
**Idempotency** means a load can be safely retried without unintentionally creating duplicate data.

Example:

```text
Run 1 → Load sales file
Run 2 → Same file arrives again
       ↓
No unwanted duplicates
```

Ways to achieve this can include:
- Tracking processed files
- Using appropriate keys
- `MERGE`
- Incremental-loading strategies
- Designing pipelines for safe retries

**Remember:**
> **Idempotency = Safe retry without duplicates**

### 6. Choosing the Right Tool

| Scenario | Best Choice |
|---|---|
| Large bulk file load | `COPY INTO` |
| Visual orchestration/scheduling | Data Pipeline |
| Transform before loading | Dataflow Gen2 |
| Complex/code-based transformation | Spark |
| Data already in supported Fabric item | Cross-item T-SQL / appropriate Fabric capability |
| Repeated automated workflow | Data Pipeline |

---

## 4.4 – Query a Data Warehouse

### 1. SQL Query Editor
- Fabric Warehouse provides a built-in SQL query editor.
- Used to write T-SQL queries.

Important SQL concepts:
- `SELECT`
- `WHERE`
- `JOIN`
- `GROUP BY`
- `ORDER BY`
- CTEs
- Window functions

**Remember:**
> **Warehouse → T-SQL-first**

### 2. T-SQL Capabilities
Fabric Warehouse provides extensive T-SQL support, including:
- Tables
- Views
- Stored procedures
- Functions where supported
- CTEs
- Window functions
- Joins
- Aggregations
- Cross-database queries

**Note:** Fabric Warehouse is not identical to SQL Server or Azure SQL Database, so some T-SQL features differ.

### 3. Cross-Database Queries
- Supported Fabric data stores can be queried together in appropriate scenarios.
- Three-part naming:

```text
Database.Schema.Table
```

Example:

```sql
SELECT
    s.OrderId,
    c.CustomerName
FROM SalesWarehouse.dbo.Orders AS s
JOIN CustomerLakehouse.dbo.Customers AS c
    ON s.CustomerId = c.CustomerId;
```

**Remember:**
> **Three-part naming = Database.Schema.Table**

### 4. Views
- A **view** is a saved query.
- Views can:
  - Simplify complex joins
  - Hide query complexity
  - Expose required columns
  - Provide a consistent interface for consumers

**Remember:**
> **View = Saved query / simplified access**

### 5. Performance
Fabric Warehouse provides performance optimizations through its underlying engine, including:
- Query optimization
- Statistics
- Data organization/optimization
- Caching mechanisms where applicable

**Remember:**
> **Warehouse → Automatic query optimization**

### 6. External Tools
Warehouse can be accessed using supported external tools and applications, such as:
- SQL Server Management Studio (SSMS)
- Azure Data Studio where supported
- Power BI
- Other supported BI/SQL tools

**Remember:**
> **Warehouse → Fabric + External SQL/BI tools**

### 7. Warehouse vs Lakehouse SQL Endpoint

#### Warehouse
- T-SQL-first
- Supports supported read/write T-SQL operations
- Structured analytical data

#### Lakehouse SQL Analytics Endpoint
- SQL access to Lakehouse tables
- Primarily read-only
- Lakehouse data can still be modified through other supported methods such as Spark, Dataflow, or Pipeline

**Remember:**
> **Warehouse → T-SQL Read + Write**

> **Lakehouse SQL Endpoint → T-SQL Query**

### 8. Important Query Concepts

#### Filtering
```sql
WHERE
```
> Select specific rows.

#### Joining
```sql
JOIN
```
> Combine related tables.

#### Aggregation
```sql
GROUP BY
SUM()
AVG()
COUNT()
```
> Summarize data.

#### CTE
```sql
WITH
```
> Create a named temporary query result for a statement.

#### Window Functions
Examples:
```sql
ROW_NUMBER()
RANK()
SUM() OVER()
```
> Perform calculations across related rows without collapsing rows like `GROUP BY`.

---

## 4.5 – Get Started with Microsoft Fabric

### 1. Licensing and Capacity

#### Fabric Trial
- Provides temporary access to Fabric capabilities for evaluation and learning.

#### Fabric Capacity
- Paid Fabric capacity provides compute resources.
- Fabric capacity uses **F SKUs**, such as F2 and higher.
- Capacity can be shared across Fabric workloads.

**Remember:**
> **Capacity = Shared compute resources**

**Important:**
- For current Fabric study, focus primarily on **F SKUs**.
- Do not make legacy Power BI Premium P SKUs a central part of your Fabric notes.

### 2. Tenant Settings
- Fabric availability is controlled through **tenant-level settings**.
- Fabric administrators can control access to Fabric capabilities.
- Tenant settings can control areas such as:
  - Fabric usage
  - Workspace creation
  - Trial access
  - Sharing
  - Specific Fabric capabilities

**Remember:**
> **Tenant Settings → Admin controls**

### 3. Workspaces
- A workspace is a container for Fabric items.
- Workspaces can be assigned to capacity.
- Workspace roles determine what users can do.

| Role | General Purpose |
|---|---|
| **Admin** | Full workspace management |
| **Member** | Manage content and collaborate |
| **Contributor** | Create/manage content |
| **Viewer** | View content |

**Remember:**
> **Workspace = Items + Collaboration + Permissions**

### 4. Domains
- Domains help organize Fabric items according to business areas.
- Useful for large organizations with many workspaces.

```text
Organization
    ↓
Domains
 ┌──────┼──────┐
Sales  Finance  HR
```

**Remember:**
> **Domain → Business-oriented grouping**

### 5. Experience Switcher
Fabric provides different experiences/workloads, including:
- Data Engineering
- Data Factory
- Data Warehouse
- Real-Time Intelligence
- Data Science
- Power BI

These workloads operate within the broader Fabric platform and can work with shared data in OneLake.

### 6. Creating Fabric Items
From a workspace, supported items can be created, such as:
- Lakehouse
- Warehouse
- Notebook
- Dataflow Gen2
- Data Pipeline
- Eventstream
- Eventhouse
- Semantic model
- Report

**Remember:**
> **Workspace → New Item → Fabric item**

### 7. OneLake Catalog
- Helps users discover and explore Fabric data and items they have access to.
- Provides a central discovery experience across Fabric.

**Remember:**
> **OneLake Catalog → Discover and explore**

### 8. Basic Fabric Setup Flow

```text
Fabric Tenant
      ↓
Tenant Settings
      ↓
Capacity
      ↓
Workspace
      ↓
Create Items
      ↓
OneLake
      ↓
Analytics
```

---

## 4.6 – Monitor a Data Warehouse

### 1. Why Monitor?
Monitoring helps understand:
- Running queries
- Query duration
- Query failures
- User/session activity
- Resource consumption
- Warehouse performance
- Capacity utilization

**Remember:**
> **Monitor → Find performance problems + failures + resource issues**

### 2. Dynamic Management Views (DMVs)
DMVs provide information about Warehouse activity and performance.

#### `sys.dm_exec_requests`
- Shows currently executing requests.
- Useful for identifying running queries.

```sql
SELECT *
FROM sys.dm_exec_requests
WHERE status = 'running';
```

**Remember:**
> `dm_exec_requests → Current requests`

#### `sys.dm_exec_sessions`
- Provides information about sessions/connections.

**Remember:**
> `dm_exec_sessions → Sessions`

#### DMV Note
- Do not assume every SQL Server DMV behaves identically in Fabric Warehouse.
- Focus on the purpose of supported DMVs.

### 3. Query Insights
- Provides information about query execution/history.
- Helps analyze query performance and query behavior.

**Remember:**
> **Query Insights → Query history + performance analysis**

### 4. Fabric Capacity Metrics
- Provides a capacity-level view of resource consumption.
- Helps understand:
  - Capacity utilization
  - Compute usage
  - CU consumption
  - Workload impact
  - Capacity pressure

```text
Fabric Capacity
      ↓
 ┌────┼──────┐
 ↓    ↓      ↓
DW  Spark  Dataflow
```

**Remember:**
> **Capacity Metrics → Capacity-wide resource usage**

### 5. Monitoring Hub
- Central place to monitor Fabric job/run activity.
- Can monitor supported activities such as:
  - Pipelines
  - Dataflows
  - Notebooks
  - Other Fabric jobs

Can show:
- Run status
- Start/end time
- Duration
- Failures
- Errors

**Remember:**
> **Monitoring Hub → Monitor Fabric job runs**

### 6. What to Look For

#### Long-Running Queries
- Queries taking unusually long.

#### Blocked Queries
- Queries waiting because of other operations/resources.

#### High Resource Consumption
- Excessive compute/CPU
- Memory/resource usage
- Capacity consumption

#### Failed Loads
- Loading operations that fail.

#### Capacity Pressure
- Heavy capacity utilization can affect workload performance.

### 7. Basic Performance Tuning

#### Query Optimization
- Reduce unnecessary processing.
- Filter appropriately.
- Avoid unnecessary joins and columns.

#### Query Plan
- Review query execution information where available.
- Identify expensive operations.

#### Views / Materialization
- Use reusable views and supported materialization approaches where appropriate.

#### Capacity Awareness
- Remember that Fabric capacity is shared.
- Other workloads can affect available resources.

**Remember:**
> **Performance = Query efficiency + Data design + Capacity awareness**

### 8. Monitoring Tools – Quick Memory

> **DMVs → What's happening now?**

> **Query Insights → What happened with queries?**

> **Monitoring Hub → What happened with Fabric jobs?**

> **Capacity Metrics → How much capacity is being used?**

---

## 4.7 – Secure a Data Warehouse

### 1. Layered Security Model

Security can be applied at different levels:

```text
Tenant
  ↓
Workspace
  ↓
Warehouse
  ↓
Objects
  ↓
Rows
  ↓
Columns / Data Visibility
```

**Remember:**
> **Higher level → Broader access**

> **Lower level → More granular access**

### 2. Workspace Roles
Main workspace roles:
- **Admin**
- **Member**
- **Contributor**
- **Viewer**

They determine what users can do within the workspace.

**Remember:**
> **Workspace role = What the user can do with workspace content**

### 3. Item-Level Permissions
- Fabric items can have permissions controlling access.
- Examples include permissions related to:
  - Read
  - Read all data
  - Build

Workspace roles and item/data permissions are different layers of access control.

**Remember:**
> **Workspace permissions + item/data permissions → Effective access**

### 4. Object-Level Security
- Object-level permissions can control access to database objects such as:
  - Tables
  - Views

Conceptually:

```sql
GRANT SELECT ON dbo.Sales TO [analyst_group];
```

**Remember:**
> **Object security → Control access to database objects**

### 5. Row-Level Security (RLS)
- RLS restricts which **rows** a user can see.
- Different users can see different rows from the same table.

Example:

```text
Sales Table
────────────────────
Region   Sales
North    1000
South    2000
West     1500
```

A North user may see only:

```text
North    1000
```

**Remember:**
> **RLS = Which rows can I see?**

### 6. Column-Level Security (CLS)
- Controls access to specific columns.

Example:

```text
Employee
────────────────────────
Name | Department | Salary
```

A user may be allowed to see:

```text
Name | Department
```

but not:

```text
Salary
```

**Remember:**
> **CLS = Which columns can I see?**

### 7. Dynamic Data Masking (DDM)
- Masks sensitive values for users who should not see the full value.
- Does not change the underlying stored value.

Example:

```text
Actual:
9876543210

Displayed:
********10
```

**Remember:**
> **DDM = Hide sensitive values**

### 8. RLS vs CLS vs DDM

| Security | Controls |
|---|---|
| **RLS** | Rows |
| **CLS** | Columns |
| **DDM** | How sensitive values are displayed |
| **Object permissions** | Tables/views/objects |
| **Workspace roles** | Workspace-level actions |

### Easy Memory

> **RLS → Rows**

> **CLS → Columns**

> **DDM → Mask values**

### 9. Power BI Security
- Warehouse data can be consumed by Power BI.
- Power BI semantic-model permissions and security should be considered separately from Warehouse permissions.

```text
Warehouse
    ↓
Semantic Model
    ↓
Power BI
```

**Remember:**
> **Warehouse security ≠ automatically identical to Power BI semantic-model security**

---

# Lesson 4 – Final Revision

| Topic | Main Idea |
|---|---|
| **4.1 Fabric Recap** | Fabric + OneLake + End-to-end analytics |
| **4.2 Warehouse** | Structured data + T-SQL |
| **4.3 Loading** | COPY INTO + Pipeline + Dataflow |
| **4.4 Querying** | T-SQL + joins + views + CTEs |
| **4.5 Fabric Setup** | Tenant + Capacity + Workspace + Items |
| **4.6 Monitoring** | DMVs + Query Insights + Capacity Metrics |
| **4.7 Security** | Workspace + Object + RLS + CLS + DDM |

# Lesson 4 in One Picture

```text
                  MICROSOFT FABRIC
                         ↓
                      OneLake
                         ↓
                     WAREHOUSE
                         ↓
              Structured Data
                         ↓
                       T-SQL
                  ↙      ↓      ↘
              Load    Query    Secure
               ↓        ↓        ↓
          COPY INTO   Views    RLS/CLS
          Pipeline    JOIN     DDM
          Dataflow    CTE
               ↓        ↓
               └────┬───┘
                    ↓
               Semantic Model
                    ↓
                 Power BI
```

# 🔑 Final Memory

> **Warehouse → Structured + T-SQL**

> **COPY INTO → Bulk Load**

> **Pipeline → Orchestrate**

> **Dataflow → Transform + Load**

> **T-SQL → Query**

> **View → Saved Query**

> **DMVs → Monitor Activity**

> **Query Insights → Query History**

> **Capacity Metrics → Capacity Usage**

> **RLS → Rows**

> **CLS → Columns**

> **DDM → Mask Values**

> **Power BI → Visualize**


