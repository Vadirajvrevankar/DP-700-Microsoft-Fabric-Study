# DP-700: Choosing the Right Transformation Tool in Microsoft Fabric

## Overview

This topic explains how to choose the right tool for transforming data in Microsoft Fabric.

The main factors are:
1. **Skill profile** — who will build and maintain the transformation.
2. **Data volume** — how much data must be processed.
3. **Transformation complexity** — whether low-code transformations are enough or custom code is needed.
4. **Data location** — where the data already lives and which tool works naturally with it.

The four tools covered here:
- **Dataflow Gen2** — low-code, analyst-friendly transformations.
- **Notebook (Spark)** — large-scale processing and custom code.
- **KQL** — time-series and event analytics, especially in Eventhouse.
- **T-SQL** — SQL-based transformations in a Fabric Warehouse.

---

## 1. Decide on Skill Profile and Data Volume First

These two factors often help narrow down the choices before considering other requirements.

### A. Skill profile

Skill profile means the technical skills of the team responsible for transforming the data.

| Team's skillset | Tool to consider |
|---|---|
| Business analyst, low-code | Dataflow Gen2 |
| Python or Spark developer | Notebook |
| KQL developer | KQL |
| SQL developer working with Warehouse data | T-SQL |

**Example:** If analysts know Excel and Power Query but not Python, Dataflow Gen2 may be a suitable choice.

### B. Data volume

Data volume means how much data the transformation needs to process.

| Data volume or workload | Tool to consider |
|---|---|
| Small datasets | Dataflow Gen2 |
| Very large datasets, such as hundreds of GB or TB | Notebook / Spark |
| High-volume telemetry and time-series events | KQL |
| Data already stored in a Warehouse | T-SQL |

**Exam tip:** Skillset and data volume are important first clues, but also consider transformation complexity and where the data is stored.

---

## 2. Dataflow Gen2

Dataflow Gen2 is a low-code data transformation tool based on Power Query. It lets you transform data through a graphical interface rather than writing large amounts of code.

### Example: Cleaning employee data

Suppose you have three Excel files:
- `Employees.xlsx`
- `Departments.xlsx`
- `Salaries.xlsx`

You need to:
1. Connect to the files.
2. Remove duplicate records.
3. Replace missing values.
4. Join employee and department data.
5. Load the cleaned data into a Lakehouse.

Dataflow Gen2 is a natural option for this kind of analyst-driven transformation.

**When to consider it**
- The team prefers low-code tools.
- The transformation uses common data-cleaning steps.
- Data comes from multiple small sources.
- Custom code is not required.

**Exam clue:** “Low-code,” “analyst,” and “many small sources” point toward Dataflow Gen2.

---

## 3. Notebook (Spark)

A Fabric notebook lets you write code, often using Python and PySpark, to transform data using Spark. It is suitable when transformations require custom logic or large-scale processing.

### Example: Processing 500 GB of sales data

Suppose you need to:
1. Read 500 GB of sales data.
2. Join it with customer data.
3. Apply complex business rules.
4. Perform fuzzy matching.
5. Write the results as Delta tables.

A notebook is a strong candidate because Spark supports distributed processing and custom code.

**When to consider it**
- The dataset is very large.
- You need custom Python or Spark logic.
- The transformation is complex.
- You need custom algorithms or machine-learning logic.

**Exam clue:** “TB-scale,” “custom code,” “Python,” or “ML” point toward a notebook/Spark.

---

## 4. Dataflow Gen2 vs. Notebook

These are two common options for general-purpose batch data transformation.

| Feature | Dataflow Gen2 | Notebook |
|---|---|---|
| Coding | Low-code | Python / PySpark / SQL |
| Best fit | Analyst-driven transformations | Complex, large-scale transformations |
| Custom logic | Limited by available transformations | Highly flexible |
| Machine learning / custom algorithms | Not the primary use case | Suitable |
| Typical exam clue | Small sources, low-code | Large data, custom code |

**Important:** A tool being technically capable of a task does not automatically make it the best answer. Match the tool to the team's skills, data volume, and requirements.

---

## 5. KQL — Time-Series and Telemetry Data

KQL stands for **Kusto Query Language**. It is designed for querying and analyzing large volumes of event and time-series data.

In Microsoft Fabric, KQL is associated with Eventhouse and its KQL databases.

### What is telemetry?

Telemetry is data generated continuously by devices, applications, or systems.

Examples:
- Temperature readings from IoT sensors.
- Website traffic events.
- Application error logs.
- Electricity meter readings.
- Server CPU usage.

### Example: Electricity meter readings

| Meter ID | Timestamp | Voltage |
|---|---|---:|
| M101 | 10:00:00 | 230 |
| M101 | 10:00:05 | 232 |
| M101 | 10:00:10 | 228 |

You need to calculate the average voltage every five minutes and identify unusual readings. KQL is suitable for this type of time-based analysis.

### What is a rolling window?

A rolling window calculates a value over a moving period of time.

Examples:
- Calculate the average temperature over the last 10 minutes.
- Count errors during the last 5 minutes.
- Track average power consumption over a moving 30-minute period.

**Exam clue:** If the question mentions telemetry, time-series data, rolling windows, or Eventhouse, consider KQL.

**Remember:** Do not choose KQL just because the problem involves SQL-like queries. Data location and workload matter.

---

## 6. T-SQL — Warehouse-Resident Transformations

T-SQL is Microsoft's SQL dialect used in SQL Server and Fabric Warehouse.

Use it when data is already in a Warehouse and the required transformation can be expressed naturally in SQL.

### Example: Update a customer table using MERGE

Suppose your Fabric Warehouse contains `Customer` and `CustomerUpdates`. You need to update existing customers while inserting new customers.

```sql
MERGE INTO dbo.Customer AS target
USING dbo.CustomerUpdates AS source
ON target.CustomerID = source.CustomerID

WHEN MATCHED THEN
    UPDATE SET target.CustomerName = source.CustomerName

WHEN NOT MATCHED THEN
    INSERT (CustomerID, CustomerName)
    VALUES (source.CustomerID, source.CustomerName);
```

### What does MERGE do?
- If the customer already exists, update the record.
- If the customer does not exist, insert a new record.

This is useful for **upsert** operations.

### What is CTAS?

CTAS means **CREATE TABLE AS SELECT**. It creates a new table using the result of a query.

```sql
CREATE TABLE dbo.HighValueCustomers
AS
SELECT CustomerID, CustomerName, TotalSales
FROM dbo.CustomerSales
WHERE TotalSales > 100000;
```

This example creates a new table containing customers whose sales exceed 100,000.

**Exam clue:** Warehouse-resident data + SQL-skilled team + `MERGE` or `CTAS` → consider T-SQL.

---

## 7. Don't Force One Tool Across the Entire Pipeline

A pipeline can use different tools at different stages. You do not have to use the same tool for every step.

### Example: Retail data pipeline

1. **Data ingestion:** Copy sales data from source systems into OneLake.
2. **Data cleaning:** Use Dataflow Gen2 to clean small datasets from multiple sources.
3. **Complex transformation:** Use a notebook to process large datasets and apply custom business logic.
4. **Warehouse loading:** Use T-SQL to load and update warehouse tables with `MERGE`.

Each tool is selected according to the needs of that particular stage.

**Exam tip:** Combining tools across a pipeline is a valid design choice, not a compromise.

---

## 8. Revisit the Decision as Data Volume Grows

The right tool can change as your data grows.

| Stage | Data volume | Possible choice |
|---|---:|---|
| Initially | 5 GB | Dataflow Gen2 for straightforward transformations |
| As it grows | 500 GB | Consider a notebook if the workload becomes resource-intensive or needs distributed processing |
| Larger scale | TB-scale | Spark notebooks may be appropriate for complex, large-scale transformations |

**Important:** These volumes are illustrative, not strict product limits. Dataflow Gen2 is not automatically unsuitable at 500 GB. The decision depends on workload complexity, performance, available capacity, and other requirements.

---

## 9. Exam Tips

- **“Low-code, many small sources, analyst”** → Dataflow Gen2.
- **“TB-scale, custom code, ML”** → Notebook / Spark.
- **“Telemetry, time-series, rolling windows”** → KQL.
- **“SQL-only team, Warehouse-resident, MERGE / CTAS”** → T-SQL.
- A tool being capable of a task does not make it the best answer. Match the skillset and data volume, not raw capability.
- Watch for scenarios that combine clues from different tools. For example, a SQL-skilled team processing 500 GB/day with custom logic may need a notebook because volume and expressiveness matter.
- Do not force a single tool across the entire pipeline.
- Reassess the tool choice as data volume and complexity grow.

---

## 10. Key Takeaways

- The transformation-tool decision depends on **skill profile, data volume, transformation expressiveness, and where the data already lives**.
- Dataflow Gen2 and notebooks both handle general batch transformation.
- KQL is scoped to Eventhouse and real-time/event workloads.
- T-SQL is suited to Warehouse-resident, SQL-shaped transformations.
- `MERGE` is a T-SQL operation used for update-and-insert (upsert) patterns.
- `CTAS` creates a table from a query result.
- Custom logic without a suitable built-in equivalent—such as fuzzy matching, machine learning, or arbitrary Python—is a strong signal to consider a notebook.

### Memory Trick: A–S–T–W

- **A — Analyst** → Dataflow Gen2
- **S — Spark-scale / custom code** → Notebook
- **T — Time-series** → KQL
- **W — Warehouse SQL** → T-SQL

---

## 11. Quick DP-700 Practice Quiz

### Question 1

An analyst needs to clean and combine three small Excel files using a low-code interface. Which tool should they consider?

A. Notebook  
B. Dataflow Gen2  
C. KQL  
D. T-SQL

**Answer: B — Dataflow Gen2.** It is designed for low-code, analyst-driven data transformation.

### Question 2

A team needs to process 2 TB of data and apply custom Python logic. Which tool is most appropriate?

A. Dataflow Gen2  
B. KQL  
C. Notebook / Spark  
D. T-SQL

**Answer: C — Notebook / Spark.** Spark notebooks support distributed processing and custom code.

### Question 3

An IoT company wants to analyze telemetry and calculate rolling averages on data stored in Eventhouse. Which tool should it use?

A. KQL  
B. T-SQL  
C. Dataflow Gen2  
D. Notebook only

**Answer: A — KQL.** KQL is designed for time-series and event analytics in Eventhouse.

### Question 4

A Fabric Warehouse contains customer data. The team needs to update existing rows and insert new rows. Which operation is appropriate?

A. CTAS  
B. MERGE  
C. Rolling window  
D. Dataflow Gen2

**Answer: B — MERGE.** MERGE supports matching source rows to update existing rows and inserting unmatched rows.

### Question 5

A team already uses Dataflow Gen2, but its data volume and transformation complexity have grown substantially. What should it do?

A. Always keep Dataflow Gen2  
B. Move everything to KQL  
C. Reassess the workload and consider a notebook  
D. Always use T-SQL

**Answer: C — Reassess the workload and consider a notebook.** Tool selection should be revisited as data volume and complexity change.

---

## One-Page Revision Summary

| Scenario | Tool / concept |
|---|---|
| Low-code analyst workflow | Dataflow Gen2 |
| Small sources and common cleaning | Dataflow Gen2 |
| Large-scale custom transformations | Notebook / Spark |
| Python, fuzzy matching, ML | Notebook / Spark |
| Telemetry and time-series | KQL |
| Eventhouse-resident analysis | KQL |
| Warehouse-resident SQL transformations | T-SQL |
| Update existing + insert new | `MERGE` |
| Create a table from a query | `CTAS` |
| Multi-stage pipeline | Use different tools where appropriate |
| Growing data volume | Reassess tool choice |

**Final exam rule:** Choose the tool that best matches the team's skills, data volume, transformation complexity, and data location—not simply the tool with the most capabilities.
