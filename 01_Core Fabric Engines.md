# Lesson 1 -- Core Fabric Engines

## 1.1 -- Ingest Data with Dataflow Gen2

### 1. What is Dataflow Gen2?

-   **Dataflow Gen2** is a Microsoft Fabric tool for **ingesting,
    transforming, and loading data**.
-   It is built on **Power Query**.
-   It provides a **low-code/no-code** approach to ETL.

**Remember:**\
`Dataflow Gen2 = Ingest + Transform + Load`

### 2. Connect to Data Sources

-   Dataflow Gen2 can connect to supported data sources.
-   Examples:
    -   Databases
    -   Files
    -   APIs
    -   Data lakes
    -   Other supported sources

**Example:**\
`SQL Database → Dataflow Gen2`

### 3. Power Query Editor

-   Dataflow Gen2 uses the **Power Query editor** for data
    transformations.
-   Transformations can be performed through a visual interface.

Common transformations: - Filter rows - Remove columns - Change data
types - Rename columns - Split columns - Merge queries - Append
queries - Remove duplicates - Group data

### 4. Transformation Steps

-   Each transformation is recorded as a **step**.
-   Steps are executed in sequence.

**Example:**\
`Source → Filter → Change Type → Remove Duplicates → Load`

### 5. M Language

-   Power Query uses the **M language**.
-   Power Query generates the underlying M code for transformations.
-   You normally don't need to manually write M code.

**Remember:**\
`Power Query → M Language`

### 6. Data Destination

-   Dataflow Gen2 loads transformed data into a destination.
-   Common destinations include:
    -   Fabric Lakehouse
    -   Fabric Warehouse
    -   Azure SQL Database
    -   Other supported destinations

**Example:**\
`SQL → Dataflow Gen2 → Lakehouse`

### 7. Compute

-   Dataflow Gen2 uses Fabric compute to process transformations.
-   **Compute processes the data.**
-   **Destination stores the processed data.**

**Think:**\
`Source → Processing → Destination`

### 8. Low-Code / No-Code

-   Dataflow Gen2 provides a visual experience for common
    transformations.
-   It reduces the need for writing complex code.

### 9. Dataflow Gen2 vs Pipeline

**Dataflow Gen2** - Data transformation and loading - Uses Power Query

**Pipeline** - Orchestration and automation - Controls execution of
activities

**Remember:**\
`Dataflow → Transform`\
`Pipeline → Orchestrate`

### 10. Dataflow Gen2 vs Spark

**Dataflow Gen2** - Low-code - Visual transformations - Power Query
based

**Spark** - Code-based - Complex transformations - Large-scale
processing

**Remember:**\
`Dataflow → Visual ETL`\
`Spark → Code/Complex Processing`

### 11. Refresh / Run

-   Dataflow Gen2 can be executed repeatedly.
-   A run reads the source, performs transformations, and loads the
    destination.
-   It can be used for recurring data workflows.

### 12. Incremental Processing

-   For large datasets, processing only new or changed data can reduce
    unnecessary processing.
-   Useful for continuously growing datasets where supported.

### 13. Typical Workflow

``` text
Connect to Source
       ↓
Transform using Power Query
       ↓
Set Destination
       ↓
Publish
       ↓
Run / Refresh
```

### 14. When to Use Dataflow Gen2?

Use Dataflow Gen2 when you need: - Low-code ETL - Power Query
transformations - Data cleaning - Data movement + transformation -
Loading data into Fabric destinations

------------------------------------------------------------------------

# 1.2 -- Orchestrate Processes and Data Movement

### 1. What is Orchestration?

-   **Orchestration** means coordinating and automating multiple
    data-processing activities.
-   Fabric **Data Pipelines** are used for orchestration.

**Remember:**\
`Pipeline = Control + Coordinate Workflow`

### 2. Why is Orchestration Important?

Real-world data workflows contain multiple steps.

**Example:**\
`Copy Data → Transform → Notebook → Notification`

A pipeline can automatically coordinate these steps.

### 3. Data Pipeline

-   A Data Pipeline provides a **visual canvas**.
-   Activities can be added and connected.
-   Activities can run sequentially or conditionally.

**Example:**\
`Copy Data → Dataflow Gen2 → Notebook`

### 4. Pipeline Activities

Activities are individual tasks inside a pipeline.

**Copy Data** - Moves data from a source to a destination.

**Dataflow** - Executes a Dataflow Gen2.

**Notebook** - Executes a Spark notebook.

**Stored Procedure** - Executes stored-procedure logic where supported.

### 5. Control Flow

Control-flow activities control how a pipeline executes.

**If Condition** - Executes different paths depending on a condition.

**For Each** - Repeats an activity for each item.

**Until** - Continues execution until a condition becomes true.

**Wait** - Pauses execution for a specified period.

**Set Variable** - Stores a value that can be used by the pipeline.

### 6. Triggers

Triggers determine **when a pipeline starts**.

**On Demand** - Pipeline is started manually.

**Scheduled** - Pipeline runs according to a schedule.

**Event-Based** - Pipeline starts when a specified event occurs.

**Remember:**\
`Trigger = When should the pipeline start?`

### 7. Parameters

-   Parameters make pipelines **reusable**.
-   Different values can be supplied without creating a new pipeline.

**Example:**\
One pipeline can process `Sales_2025.csv` and `Sales_2026.csv`.

### 8. Dynamic Content

-   Dynamic content allows values to be determined or referenced
    dynamically during execution.
-   Useful for:
    -   File names
    -   Folder paths
    -   Dates
    -   Table names
    -   Environment-specific values

### 9. Dependencies

-   Activities can depend on the result of previous activities.

**Example:**\
`Copy Data → Success → Dataflow Gen2`

### 10. Error Handling

-   Pipelines can have different paths for successful and failed
    activities.

**Example:**\
`Copy Data → Failed → Error Handling / Notification`

### 11. Monitoring

Pipeline runs can be monitored for: - Status - Duration - Activity
execution - Errors - Failures - Run details

**Remember:**\
`Monitoring = Check Pipeline Execution`

### 12. Typical Pipeline Workflow

``` text
Trigger
   ↓
Copy Data
   ↓
Dataflow Gen2
   ↓
Notebook
   ↓
Success / Failure Handling
```

### 13. When to Use Pipeline?

Use pipelines when you need: - Orchestration - Scheduling -
Dependencies - Multiple activities - Automation - Error handling

------------------------------------------------------------------------

# 1.3 -- Use Apache Spark in Microsoft Fabric

### 1. What is Apache Spark?

-   **Apache Spark** is a distributed processing engine.
-   It is designed for **large-scale data processing**.

It can be used for: - Data engineering - Data transformation - Data
analysis - Data science - Machine learning

**Remember:**\
`Spark = Distributed Data Processing`

### 2. Where Does Spark Run?

In Fabric, Spark workloads can be developed/executed using:

**Notebooks** - Mainly for interactive development and exploration.

**Spark Job Definitions** - Used to execute Spark jobs/scripts as
repeatable workloads.

**Remember:**\
`Notebook → Interactive`\
`Spark Job Definition → Job Execution`

### 3. Spark Languages

Fabric Spark supports languages including: - **PySpark / Python** -
**Spark SQL** - **Scala** - **R**

### 4. Spark Session

-   A Spark session is the entry point for interacting with Spark.
-   It allows Spark code to:
    -   Read data
    -   Transform data
    -   Write data
    -   Execute Spark operations

**Think:**\
`Notebook → Spark Session → Spark Processing`

### 5. Spark Compute

-   Spark requires compute resources to process data.
-   Fabric provides Spark compute for Spark workloads.

**Simple idea:**\
`Spark Code → Compute → Data Processing`

### 6. Starter Pools

-   Starter Pools provide **preconfigured resources for faster Spark
    startup**.
-   They are useful for interactive notebook work.

**Remember:**\
`Starter Pool → Faster Startup → Interactive Work`

### 7. Custom Spark Pools

-   Custom pools allow you to configure Spark resources according to
    workload requirements.
-   Configuration can include:
    -   Node size/type
    -   Number of nodes
    -   Autoscaling-related settings
    -   Other Spark settings

**Remember:**\
`Custom Pool → More Control`

### 8. DataFrames

-   A Spark DataFrame is a **distributed tabular data structure**.
-   It is conceptually similar to a Pandas DataFrame but designed for
    distributed processing.

**Remember:**\
`Spark DataFrame = Distributed Table-Like Data`

### 9. Spark vs Pandas

**Pandas** - Primarily processes data on a single machine.

**Spark DataFrame** - Designed for distributed processing across Spark
resources.

### 10. Reading Data

-   Spark can read data from supported storage systems.
-   An important Fabric storage location is **OneLake**.

**Example:**\
`OneLake → Spark → DataFrame`

### 11. Writing Data

-   Spark can write processed data back to storage.

**Example:**\
`Spark → Transform → Delta Table → OneLake`

### 12. Delta Lake

-   Delta Lake is an important table/storage format in Fabric Lakehouse.
-   Spark can read and write Delta tables.
-   Lakehouse tables use Delta format.

**Remember:**\
`Spark = Processing`\
`Delta = Table Format`\
`OneLake = Storage`

### 13. Notebook + Pipeline Integration

A Spark notebook can be executed as an activity in a Data Pipeline.

**Example:** `Pipeline → Copy Data → Dataflow Gen2 → Notebook`

-   Pipeline → Orchestration
-   Notebook → Spark processing

### 14. Spark Job Definition + Pipeline

A Spark Job Definition can also be used as part of an automated
workflow.

**Example:**\
`Pipeline → Spark Job Definition → Spark Processing → Lakehouse`

------------------------------------------------------------------------

# 1.4 -- Work with Real-Time Data in an Eventhouse

### 1. What is an Eventhouse?

-   An **Eventhouse** is a Microsoft Fabric item designed for
    **real-time analytics**.
-   It is optimized for:
    -   Streaming data
    -   Event data
    -   Time-series data
    -   High-volume/high-velocity data

**Remember:**\
`Eventhouse = Store + Analyze Real-Time Data`

### 2. Why Eventhouse?

-   Lakehouse and Warehouse support many analytical workloads.
-   Eventhouse is specifically optimized for **real-time event and
    time-series analytics**.
-   It is designed for continuously arriving data.

**Remember:**\
`Lakehouse → Data Engineering & Analytics`\
`Warehouse → SQL Analytics`\
`Eventhouse → Real-Time Analytics`

### 3. KQL

-   **KQL = Kusto Query Language**.
-   It is used to query data in Eventhouse.
-   KQL is also used by **Azure Data Explorer**.
-   It is designed for analyzing large volumes of event/log data.

**Remember:**\
`Eventhouse → KQL`

### 4. KQL Database

-   An Eventhouse can contain **KQL databases**.
-   KQL databases contain tables that store event data.

``` text
Eventhouse
    ↓
KQL Database
    ↓
Tables
    ↓
Event Data
```

### 5. KQL Tables

Tables contain the actual event data.

Example:

  DeviceID     Temperature Timestamp
  ---------- ------------- -----------
  D101                28.5 10:01
  D102                30.2 10:02
  D101                29.1 10:03

### 6. Data Ingestion

Eventhouse can receive data through supported ingestion methods.

A common Fabric architecture is:

`Eventstream → Eventhouse`

### 7. Eventstream

-   **Eventstream** is used to ingest and process streaming events.
-   It can route streaming data to Eventhouse.
-   It can perform supported transformations on streaming data.

**Remember:**\
`Eventstream → Streaming Ingestion/Processing`\
`Eventhouse → Store + Analyze Real-Time Data`

### 8. Batch / Manual Ingestion

-   Eventhouse can also receive data through supported batch/manual
    ingestion methods.
-   This can be useful for loading historical data.

### 9. Update Policies

-   Update policies can automatically transform data when new data
    arrives.
-   Data can be ingested into one table and transformed into another
    table.

``` text
Incoming Data
     ↓
Source Table
     ↓
Update Policy
     ↓
Transformed Table
```

**Remember:**\
`Update Policy → Transform Incoming Data`

## KQL Key Concepts

### 10. `where`

-   Filters rows based on a condition.

``` kusto
Temperature
| where Temperature > 30
```

**Remember:**\
`where → Filter`

### 11. `project`

-   Selects the columns you want in the result.

``` kusto
Temperature
| project DeviceID, Temperature
```

**Remember:**\
`project → Select Columns`

### 12. `summarize`

-   Performs aggregation.
-   Similar conceptually to `GROUP BY` in SQL.

``` kusto
Temperature
| summarize AvgTemp = avg(Temperature) by DeviceID
```

**Remember:**\
`summarize → Aggregate`

### 13. `bin()`

-   Groups values into fixed-size intervals.
-   Particularly useful for time-series analysis.

``` kusto
Temperature
| summarize count() by bin(Timestamp, 1h)
```

**Remember:**\
`bin() → Time/Data Buckets`

### 14. Real-Time Query Performance

-   Eventhouse/KQL is designed for fast querying of large volumes of
    event data.
-   It is optimized for high-volume, high-velocity real-time analytics.
-   Actual performance depends on the workload, data, query, and
    capacity.

### 15. Real-Time Dashboard

Eventhouse data can be used in **Real-Time Dashboards**.

``` text
IoT Devices
     ↓
Eventstream
     ↓
Eventhouse
     ↓
KQL Query
     ↓
Real-Time Dashboard
```

### 16. Activator

-   **Fabric Activator** can monitor data for specified conditions.
-   When a condition is detected, it can trigger an action.

**Example:**

``` text
Temperature > 80°C
       ↓
Activator
       ↓
Trigger Action
```

------------------------------------------------------------------------

# Lesson 1 -- Final Revision

  Concept                 Main Purpose
  ----------------------- ---------------------------------------
  **Dataflow Gen2**       Ingest + Transform + Load
  **Data Pipeline**       Orchestration + Automation
  **Apache Spark**        Distributed processing
  **Eventhouse**          Real-time event analytics
  **Power Query**         Visual data transformation
  **M Language**          Power Query language
  **Pipeline Activity**   Individual pipeline task
  **Trigger**             Starts pipeline
  **DataFrame**           Distributed tabular data
  **OneLake**             Fabric's central storage
  **Delta**               Lakehouse table format
  **Eventstream**         Streaming ingestion/processing
  **KQL**                 Query Eventhouse data
  **Update Policy**       Transform incoming Eventhouse data
  **Activator**           Detect conditions and trigger actions

## 🧠 4 Things to Remember

``` text
Dataflow Gen2
    ↓
Transform


Pipeline
    ↓
Orchestrate


Spark
    ↓
Distributed Processing


Eventhouse
    ↓
Real-Time Analytics
```


